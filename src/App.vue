<template>
  <div id="app">
    <BarcodeScanner />
  </div>
</template>

<script>
import BarcodeScanner from './components/BarcodeScanner.vue';

export default {
  name: 'app',
  components: {
    BarcodeScanner
  },
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
