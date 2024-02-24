<template>

  <div id="app">
    <VideoRecorder />
  </div>
</template>

<script>

import VideoRecorder from './components/VideoRecorder.vue';
export default {
  name: 'App',
  components: {
    
    VideoRecorder
  }
}
</script>

<style>
#app {
  font-family: Arial, sans-serif;
  text-align: center;
  margin-top: 20px;
}
</style>
