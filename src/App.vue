<template>
  <HelloWorld />
  <PointCloudView />
</template>

<script setup>
import HelloWorld from './components/HelloWorld.vue'
import PointCloudView from './components/PointCloudView.vue';
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

#app {
  width: 100vw;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f5f5f5;
}
</style>