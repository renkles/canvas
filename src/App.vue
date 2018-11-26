<template>
  <div class="app">
    <canvas-bg></canvas-bg>
  </div>
</template>

<script>
import canvasBg from './components/canvasBg'

export default {
  name: 'app',
  components: {
    canvasBg
  }
}
</script>

<style lang="less">
  html {
    width: 100%;
    height: 100%;
  }
  body {
    width: 100%;
    height: 100%;
    margin: 0;
    padding: 0;
    font-size: 14px;
    font-family: "Arial","Microsoft YaHei";
  }
  .app {
    width: 100%;
    height: 100%;
    overflow: hidden;
  }
</style>
