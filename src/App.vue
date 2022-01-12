<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <FractalImage :width=600 :height=600 image-id="fractal-image-0" style="border: 5px; color: black"/>
  </div>
</template>

<script>
import FractalImage from './components/FractalImage'
export default {
  name: 'App',
  components: {
    FractalImage
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
  margin-top: 60px;
}
</style>
