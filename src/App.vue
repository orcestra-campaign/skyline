<script setup>
import { ref } from 'vue';
//import { RouterLink, RouterView } from 'vue-router'
import CrossSection from './components/CrossSection.vue'
import Map from './components/Map.vue'
import ManualPathEntry from './components/ManualPathEntry.vue'
import VarSelector from './components/VarSelector.vue'
import PathInfo from './components/PathInfo.vue';

const path = ref([ [ -25, 16 ], [ -22, -2 ] ]);
const variable = ref("cc");
const ti = ref(0);
const error = ref(null);

window.addEventListener('error', (message, url, lineNumber) => {
  error.value = "SERR: " + message + " @ " + url + ":" + lineNumber;
});

window.addEventListener('unhandledrejection', (event) => {
  error.value = "AERR:" + event.reason.message + " @ " + event.reason.fileName + ":" + event.lineNumber;
});

const dataset = "https://swift.dkrz.de/v1/dkrz_948e7d4bbfbb445fbff5315fc433e36a/hera5/v2/hera5_PT1H_hpz7.zarr";

const onPathChange = (newPath) => {
  console.log("new path:", newPath);
  path.value = newPath;
};

const onVarSelect = (newVar) => {
  console.log("new var:", newVar);
  variable.value = newVar;
};

const onTimeSelect = (newTime) => {
  ti.value = Number(newTime);
};
</script>

<template>
  <!--
  <header>
    <img alt="Vue logo" class="logo" src="@/assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <HelloWorld msg="You did it!" />

      <nav>
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/about">About</RouterLink>
      </nav>
    </div>
  </header>
  -->
  <Map @path-change="onPathChange"></Map>
  <!--<ManualPathEntry @path-change="onPathChange"></ManualPathEntry>-->
  <div class="statusline">
    <VarSelector :store="dataset" @var-select="onVarSelect" @time-select="onTimeSelect"></VarSelector>
    <PathInfo :path="path"></PathInfo>
  </div>
  <CrossSection :path="path" :variable="variable" :timeIndex="ti"></CrossSection>
  <div v-if="error">{{ error }}</div>
</template>

<style scoped>
div.statusline {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}
</style>
