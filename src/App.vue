<template>
  <div id="app">
    <ImageLrz />
  </div>
</template>

<script>
import ImageLrz from './components/imageLrz.vue'

export default {
  name: 'App',
  components: {
    ImageLrz
  }
}
</script>

<style lang="less">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
