<script setup>
import { ref, guardReactiveProps, watch, onMounted } from "vue";
import "leaflet/dist/leaflet.css";
import L from "leaflet";

import "@geoman-io/leaflet-geoman-free";
import "@geoman-io/leaflet-geoman-free/dist/leaflet-geoman.css";

const map_container = ref(null);

const emit = defineEmits(["pathChange"]);

onMounted(() => {
  const map = L.map(map_container.value, {
    center: [12, -25],
    zoom: 6
  });
  L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19,
    attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
  }).addTo(map);

  map.pm.addControls({  
    position: 'topleft',
    drawMarker: false,
    drawCircleMarker: false,
    drawRectangle: false,
    drawPolygon: false,
    rotateMode: false,
  });
  map.on("pm:create", (e) => {
    console.log(e.layer.toGeoJSON().geometry.coordinates);
    emit("pathChange", e.layer.toGeoJSON().geometry.coordinates);
  });
});
</script>

<template>
  <div class="map">
    <div class="map_container" ref="map_container"></div>
  </div>
</template>

<style scoped>

.map_container {
  margin: 0 auto;
  width: 100%;
  height: 100%;
}

</style>
