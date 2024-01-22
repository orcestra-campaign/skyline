<script setup>
import CrossSection from '../components/CrossSection.vue'

const props = defineProps({
  path: {
    type: Object,
    required: true
  },
});

const lon_min = defineModel("lon_min", {default: -25, type: Number});
const lon_max = defineModel("lon_max", {default: -22, type: Number});

const lat_min = defineModel("lat_min", {default: 16, type: Number});
const lat_max = defineModel("lat_max", {default: -2, type: Number});

const variable = defineModel("variable", {default: "cc", type: String});
</script>

<template>
  <div class="about">
    <div>
      variable: <input v-model="variable" /> <br/>
      <!--lat: <input v-model="lat_min" /> -  <input v-model="lat_max" /> <br/>
      lon: <input v-model="lon_min" /> -  <input v-model="lon_max" /> <br/>-->
    </div>
    <CrossSection :path="props.path" :variable="variable">
    </CrossSection>
  </div>
</template>

<style scoped>
.about {
  display: block;
  height: max-content;
}

</style>
