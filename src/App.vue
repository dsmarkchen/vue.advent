<template>
  <div id="app">    
    <Advent>   </Advent>  
  </div>
</template>

<script>

import Advent from './components/Advent.vue'


export default {
  name: 'App',
  components: {
    Advent,
  },
  
}
</script>

<style>
@import url(//fonts.googleapis.com/css?family=Open+Sans:400,300,600);
@import url(//fonts.googleapis.com/css?family=Roboto:400,100,300,500,700);
@import url(//fonts.googleapis.com/css?family=Volkhov:400italic);

#app {
  font-family: 'Open Sans', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
