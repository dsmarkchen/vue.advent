<template>
  <div id="app">
    <Advent msg="Welcome to Adventure!!" 
            info="You are standing at the end of a road before a small brick building." />
  </div>
</template>

<script>
import Advent from './components/Advent.vue'

export default {
  name: 'App',
  components: {
    Advent
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
