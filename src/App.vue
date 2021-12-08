<template>
  <div id="app">    
    <Advent>
    </Advent>  
  </div>
</template>

<script>
import Advent from './components/Advent.vue'

export default {
  name: 'App',
  components: {
    Advent,
  },
  methods: {
    changeInfo(val) {
      console.log("info" + val);
    }
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
