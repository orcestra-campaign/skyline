<script setup>
import { ref, computed, compile } from "vue";
import ManualPathEntry from "./ManualPathEntry.vue";
import { clamp, clearTileCache } from "plotly.js-dist";

const props = defineProps({
  path: {
    type: Object,
    required: true
  },
})

function formatLonLatRte(lon, lat) {
  if (lon > 180) {
    lon = lon - 360;
  }
  const ns = lat < 0 ? "S" : "N";
  const ew = lon < 0 ? "W" : "S";
  lat = Math.abs(lat);
  lon = Math.abs(lon);
  return String(Math.floor(lat * 100)).padStart(4, "0") + ns + String(Math.floor(lon * 100)).padStart(5, "0") + ew;
}

const rte = computed(() => {
  if (props.path.length == 0) {
    return "";
  }
  let res = formatLonLatRte(props.path[0][0], props.path[0][1]);
  for ( const [lon, lat] of props.path.slice(1)) {
    res += " DCT " + formatLonLatRte(lon, lat);
  }
  return res;
});

const geojson = computed(() => JSON.stringify({
  type: "Feature",
  geometry: {
    type: "LineString",
    coordinates: props.path
  },
  properties: {}
}));

function copyToClipboard(text) {
  navigator.clipboard.writeText(text);
}

</script>


<template>
  <div class="pathinfo">
    <div>
      RTE:&nbsp;
    </div>
    <div class="rte">
    {{ rte }}
    </div>
    <div>
      &nbsp;to clipboard:
      <button @click="copyToClipboard(rte)">RTE</button>
      <button @click="copyToClipboard(geojson)">GeoJSON</button>
    </div>
  </div>
</template>

<style scoped>
div.pathinfo {
  display: flex;
  flex-direction: row;
}

div.rte {
  font-family: 'Courier New', Courier, monospace;
  flex-shrink: 1;
  padding-top: 3px;
}

</style>