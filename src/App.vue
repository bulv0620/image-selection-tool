<template>
  <div id="app">
    <ImageSelectionTool></ImageSelectionTool>
  </div>
</template>

<script>
import ImageSelectionTool from './components/ImageSelectionTool/index.vue'

export default {
  name: 'App',
  components: {
    ImageSelectionTool
  }
}
</script>

<style>
#app {
  width: 100vw;
  height: 100vh;
}
</style>
