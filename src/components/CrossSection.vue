<script setup>
import { ref, watch } from "vue";
import * as zarr from "zarrita";
import * as healpix from '@hscmap/healpix';
import Plotly from 'plotly.js-dist';

const plot_container = ref(null);

const store = new zarr.FetchStore("https://swift.dkrz.de/v1/dkrz_948e7d4bbfbb445fbff5315fc433e36a/hera5/v2/hera5_P1M_hpz7.zarr");

const props = defineProps({
  path: {
    type: Object,
    required: true
  },
  variable: {
    type: String,
    required: false,
    default: "cc",
  }
})

console.log("created with", props);

function npix2nside(npix) {
  return Math.sqrt(npix / 12);
}

function linspace(start, stop, num, endpoint = true) {
    const div = endpoint ? (num - 1) : num;
    const step = (stop - start) / div;
    return Array.from({length: num}, (_, i) => start + step * i);
}

function groupBy(xs, key) {
  return xs.reduce(function(rv, x) {
    (rv[x[key]] = rv[x[key]] || []).push(x);
    return rv;
  }, {});
};

const fetchData = async () => {
  const itime = 0;
  const nx = 100;
  const variable = props.variable;

  const root = await zarr.open(store, { kind: "group" });
  const arr = await zarr.open(root.resolve(variable), {kind: "array"});
  const level_promise = zarr.open(root.resolve("level"), {kind: "array"}).then(a => zarr.get(a, [null]));
  const dim2axis = Object.fromEntries(arr.attrs._ARRAY_DIMENSIONS.map((k, i) => [k, i]));
  console.log(arr);
  console.log(dim2axis);
  const nside = npix2nside(arr.shape[dim2axis["cell"]]);
  console.log("nside:", nside, " order:", healpix.nside2order(nside));

  // WARNING: this is a loxodrome
  const lons = linspace(props.path[0][0], props.path[1][0], nx);
  const lats = linspace(props.path[0][1], props.path[1][1], nx);

  console.log(lons, lats);
  if (lons.every(Number.isFinite) && lons.every(Number.isFinite)) {
    console.log("ok, going ahead");
  } else {
    console.log("broken coords");
    return;
  }

  const thetas = lats.map(lat => (90 - lat) * (Math.PI / 180));
  const phis = lons.map(lon => (lon % 360) * (Math.PI / 180));

  const ipix = thetas.map((theta, i) => healpix.ang2pix_nest(nside, theta, phis[i]));
  //const nx = ipix.length;
  const pixchunks = ipix.map((pix, i) => [
                                  (pix / arr.chunks[dim2axis["cell"]]) >> 0,
                                  pix % arr.chunks[dim2axis["cell"]],
                                  i]);

  const pixbychunk = groupBy(pixchunks, 0);

  const timechunk = (itime / arr.chunks[dim2axis["time"]]) >> 0;
  const timesubchunk = itime % arr.chunks[dim2axis["time"]];
  const levelchunks = Array.from({length: Math.ceil(arr.shape[dim2axis["level"]] / arr.chunks[dim2axis["level"]])},
                                 (_, i) => i);


  console.log(ipix, pixchunks, pixbychunk, timechunk, levelchunks);

  const ny = arr.shape[dim2axis["level"]];
  const cy = arr.chunks[dim2axis["level"]];
  let csdata = Array.from({length: nx}, _ => new Float32Array(ny));
  let promises = [];
  for (let [cellchunk, selection] of Object.entries(pixbychunk)) {
    for (const levelchunk of levelchunks) {
      const sel = {time: timechunk, level: levelchunk, cell: cellchunk};
      const idx = arr.attrs._ARRAY_DIMENSIONS.map(dim => sel[dim]);
      promises.push(arr.getChunk(idx).then(chunk => {
        const offset = timesubchunk * chunk.stride[dim2axis["time"]];
        const step = chunk.stride[dim2axis["level"]];

        for (const sel of selection) {
          const celloffset = sel[1] * chunk.stride[dim2axis["cell"]];
          const ix = sel[2];
          for (let levelsubchunk = 0; levelsubchunk < cy; ++levelsubchunk) {
            const iy = levelsubchunk + levelchunk * cy;
            csdata[ix][iy] = chunk.data[offset + celloffset + levelsubchunk * step];
          }
        }
      }));
    }
    //console.log(cellchunk, selection);
  }
  await Promise.all(promises);
  console.log(csdata);
  const levels = Array.from((await level_promise).data).map(l => Number(l));
  console.log("levels", levels);
  return {
    variable: variable,
    data: {
      x: lats,
      y: levels,
      z: csdata,
      type: "heatmap",
      transpose: true,
    }
  };
};

function render() {
  fetchData().then(data => {
  const layout = {
    //title: "crosssection " + data.variable,
    xaxis: {
      title: "latitude",
    },
    yaxis: {
      autorange: "reversed",
      title: "pressure",
    },
    margin: {
      t: 10,
      b: 40,
      l: 60,
      r: 60,
    },
  }
  console.log(data);
  Plotly.newPlot(plot_container.value, [data.data], layout);});
}

render();


watch(props, async (newProps, oldProps) => {
  console.log(oldProps, "->", newProps);
  render();
})
</script>

<template>
  <div ref="plot_container" class="crosssection">
  </div>
</template>

<style scoped>

</style>
