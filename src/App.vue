<template>
  <div id="app">
    <EjemploClase />

  </div>
</template>

<script>
import EjemploClase from './components/EjemploClase.vue';

export default {
  name: 'App',
  components: {
    EjemploClase

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
