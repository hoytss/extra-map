<template>
  <div id="app">
    <ControllerMap />
  </div>
</template>

<script>
import ControllerMap from './components/controllerMap.vue'

export default {
  name: 'App',
  components: {
    ControllerMap
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
