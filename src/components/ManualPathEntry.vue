<script setup>
import { watch } from "vue";

const emit = defineEmits(["pathChange"]);

const lon_min = defineModel("lon_min", {default: -25, type: Number});
const lon_max = defineModel("lon_max", {default: -22, type: Number});

const lat_min = defineModel("lat_min", {default: 16, type: Number});
const lat_max = defineModel("lat_max", {default: -2, type: Number});

watch([lon_min, lon_max, lat_min, lat_max], ([lon_min, lon_max, lat_min, lat_max]) => {
  emit("pathChange", [[Number(lon_min), Number(lat_min)], [Number(lon_max), Number(lat_max)]]);
})

</script>

<template>
  <div class="pathinput">
    lat: <input v-model="lat_min" /> -  <input v-model="lat_max" /> <br/>
    lon: <input v-model="lon_min" /> -  <input v-model="lon_max" /> <br/>
  </div>
</template>