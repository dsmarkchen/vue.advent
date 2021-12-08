<template>
  <div >
    
      <h1>{{msg }} {{author.firstName}}</h1>
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
function Person(firstName, lastName) {
  this.firstName = firstName
  this.lastName = lastName
}



function FSM(initState, transactions)
{
  this.initState = initState
  this.transactions = transactions
}



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
        fsm : new FSM('', []),
        author: new Person('', ''),
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
      this.author = new Person("Mark", "Chen");
      this.fsm = new FSM("road", {
         road: { 
           in: 'house' ,
           look: 'road'
         },
         house: {
           out: 'road',
           look: 'house'
          }
      });
  },
  methods: {    
    go() {
        console.log("act:" + this.act);
       
        let location = JSON.stringify(this.Locations[this.current]);
        console.log(location);
        console.log("author: " + this.author);
        console.log ("fsm" + this.fsm);
        var p = this.fsm.transactions;
        var initState = this.fsm.initState;
        for (var key of Object.keys(p)) {
           
          for(var child_key of Object.keys(p[key])) {
            console.log(key + " -> " + child_key + " -> " + p[key][child_key]);
          }
          
        }
        console.log("initState: " + initState);

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
