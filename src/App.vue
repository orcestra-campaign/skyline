<script setup>
import { ref } from 'vue';
//import { RouterLink, RouterView } from 'vue-router'
import CrossSection from './components/CrossSection.vue'
import Map from './components/Map.vue'
import ManualPathEntry from './components/ManualPathEntry.vue'
import VarSelector from './components/VarSelector.vue'

const path = ref([ [ -25, 16 ], [ -22, -2 ] ]);
const variable = ref("cc");

const dataset = "https://swift.dkrz.de/v1/dkrz_948e7d4bbfbb445fbff5315fc433e36a/hera5/v2/hera5_P1M_hpz7.zarr";

const onPathChange = (newPath) => {
  console.log("new path:", newPath);
  path.value = newPath;
};

const onVarSelect = (newVar) => {
  console.log("new var:", newVar);
  variable.value = newVar;
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
  <VarSelector :store="dataset" @var-select="onVarSelect"></VarSelector>
  <CrossSection :path="path" :variable="variable"></CrossSection>
</template>

<style scoped>

</style>
