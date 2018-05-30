<template>
  <div id="app">
    <img src="./assets/logo.png">
    <Yaml2Uml/>
  </div>
</template>

<script>
import Yaml2Uml from './components/Yaml2Uml.vue'

export default {
  name: 'app',
  components: {
    Yaml2Uml
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
