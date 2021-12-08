<template>
  <div class="advent">
    
      <h1>{{msg }}</h1>
      <Location></Location>
      <p> {{info}}</p>
      

      <b-form-input v-model="act"></b-form-input>
      <b-button @click="go"> enter </b-button>

      <StateMachine></StateMachine>
  </div>
</template>

<script>
import StateMachine from './StateMachine.vue'
import Location from './Location.vue'

//var Nanocomponent = require('nanocomponent')
///var nanostate = require('nanostate')

export default {
  name: 'Advent',  
  components: {
    StateMachine,
    Location    
  },  
  data() {
    return {
        current: 1,
        act: 'in',
        msg: 'welcome',
        info: 'You are standing at the end of a road before a small brick building.',
        Actions: ['in', 'out'],
        Locations:[
          {
            name:'road',
            description: "You are standing at the end of a road before a small brick building."
          },
          {
            name:'house',
            description:"You are in a house"
          }
        ]    
    };
  },
  created: function() {
     
  },
  methods: {
    go() {
        console.log("act:" + this.act);
        let location = JSON.stringify(this.Locations[this.current]);
        console.log(location);
        //console.log(this.machine);
        //this.machine.run(this.act);
        //this.info = this.Locations[this.current].description;
        this.info = (this.Locations[this.current].description);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
