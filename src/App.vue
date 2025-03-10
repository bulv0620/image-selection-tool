<template>
  <div id="app">
    <ImageSelectionTool :boxes="boxes"></ImageSelectionTool>
  </div>
</template>

<script>
import ImageSelectionTool from './components/ImageSelectionTool/index.vue'

export default {
  name: 'App',
  components: {
    ImageSelectionTool
  },
  data() {
    return {
      /**
       * box对象
       * {
       *  id:"uuid" // 唯一标识uuid
       *  name:"选框1" // 选框名称
       *  height:"245.83" // 选框高度
       *  width:"492" // 选框宽度
       *  x:"1809.72" // 选框x坐标
       *  y:"561.20" // 选框y坐标
       * }
       */
      boxes: [
      ]
    }
  }
}
</script>

<style>
#app {
  width: 100vw;
  height: 100vh;
}
</style>
