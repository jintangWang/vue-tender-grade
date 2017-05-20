<template>
  <div id="app">
    <grade></grade>
  </div>
</template>

<script>
import Grade from './components/Grade'

export default {
  name: 'app',
  components: {
    Grade
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
