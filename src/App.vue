<script setup>
import { RouterLink, RouterView } from 'vue-router'
// import HelloWorld from './components/HelloWorld.vue'
import Map from './components/Map.vue';
</script>

<template>


  <RouterView />
</template>

<style scoped>
body { margin: 0; padding: 0; }
        html, body, #map { height: 100%; }

</style>
