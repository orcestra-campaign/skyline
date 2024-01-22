<script setup>
import { ref } from 'vue';
//import { RouterLink, RouterView } from 'vue-router'
import CrossSection from './components/CrossSection.vue'
import Map from './components/Map.vue'
import ManualPathEntry from './components/ManualPathEntry.vue'

const path = ref([ [ -25, 16 ], [ -22, -2 ] ]);

const onPathChange = (newPath) => {
  console.log("new path:", newPath);
  path.value = newPath;
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
  <CrossSection :path="path"></CrossSection>
</template>

<style scoped>

</style>
