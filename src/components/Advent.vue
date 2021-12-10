<template>
  <div class="container">
    <div class="advent">
      <div class="block">
        <h1>
          Welcome
          <mark v-if="author.firstName != ''">{{ author.firstName }}</mark> to
          Adventure!!
        </h1>

        <ALocation
          :name="loc.name"
          :description="loc.description"
          :short="loc.short"
          :showDesc="full"
        ></ALocation>

        <ul>
          <div v-for="item in AObjects" :key="item.name">
            <li v-if="(item.location == loc.name) && (item.locked != undefined)">
              <AObject
                :name="item.name"
                :desc="item.desc"
                :notes="item.notes"
                :taken="item.taken"
                :locked="item.locked"
                :current="item.current"
              ></AObject>
            </li>

            <li v-if="(item.location == loc.name) && (item.taken != undefined)">
              <AObject
                :name="item.name"
                :desc="item.desc"
                :notes="item.notes"
                :taken="item.taken"
                :locked="item.locked"
                :current="item.current"
              ></AObject>
            </li>
          </div>
        </ul>

        <Score :score="score" :showscore="showscore"> </Score>

        <div>
          <b-form-group
            :description="instruction"
            label=""
            label-for="input-1"
            valid-feedback=""
            :invalid-feedback="input_invalid"
            :state="input_state"
          >
            <b-form-input id="input-1" v-model="act" v-on:keyup.enter="next" trim></b-form-input>
          </b-form-group>
        </div>

       
      </div>
    </div>
  </div>
</template>

<script>
import ALocation from "./ALocation.vue";
import Score from "./Score.vue";
import AObject from "./AObject.vue";
import objects from "../json/objects.json";
import locations from "../json/locations.json";
import transitions from "../json/transitions.json";
import vocabulary from "../json/vocabulary.json";

function Person(firstName, lastName) {
  this.firstName = firstName;
  this.lastName = lastName;
}

function FSM(initState, transitions) {
  this.initState = initState;
  this.transitions = transitions;
  this.state = initState;
  this.dump = function () {
    var p = this.transitions;
    for (var key of Object.keys(p)) {
      for (var child_key of Object.keys(p[key])) {
        console.log(key + " -> " + child_key + " -> " + p[key][child_key]);
      }
    }
  };
  this.transition = function (action) {
    var currentState = this.state;
    var p = this.transitions;
    for (var key of Object.keys(p)) {
      if (key == currentState) {
        for (var child_key of Object.keys(p[key])) {
          if (child_key == action) {
            this.state = p[key][child_key];
            break;
          }
        }
      }
    }
  };
}

export default {
  name: "Advent",
  components: {
    ALocation,
    AObject,
    Score,
  },
  data: function () {
    return {
      instruction: "Let us know what you wanna do",
      score: 0,
      showscore: false,
      showObject: false,

      fsm: new FSM("", []),
      author: new Person("", ""),
      act: "in",
      loc: null,
      AObjects: [],      
      Actions: ["in", "out"],
      Locations: locations,
    };
  },
  computed: {
    input_state() {
      let dict = vocabulary;
      let found = false;
      let words = this.act.split(" ");
      words.forEach((word) => {
        for (var key of Object.keys(dict)) {
          if (dict[key].indexOf(word) >= 0) {
            found = true;
            break;
          }
        }
        if (found == false) {
          return false;
        }
      });
      return found;
    },
    input_invalid() {
      if (this.act.length > 0) {
        return "Enter a meaningful word.";
      }
      return "Please enter something.";
    },
  },
  created: function () {
    this.showObject = false;
    this.score = 20;
    this.full = true;
    this.author = new Person("", "");
    this.AObjects = objects;
    
    this.fsm = new FSM("road", transitions);
    this.loc = JSON.parse(JSON.stringify(this.Locations[0]));
  },
  methods: {
    next() {
      console.log("act:" + this.act);
      if (this.act == "score" || this.act == "quit") {
        this.showscore = true;
        return;
      }
      if (this.act.startsWith("take") || this.act.startsWith("drop")) {
        var objname = this.act.slice(4).trim();
        console.log("take: " + objname);
        for (const aobj of this.AObjects) {
          if (aobj.name == objname) {
            aobj.taken = true;
            break;
          }
        }
        return;
      }
      if(this.act.startsWith("open")) {
        var obj2name = this.act.slice(4).trim();
        console.log("open: " + obj2name);

        // check keys
        if(obj2name == "grate") { 
          for (const aobj of this.AObjects) {
            if (aobj.name == "keys") {
              if(aobj.taken == false) {
                console.log("no keys");
                this.instruction = "no keys";
                return;
              }
            }
          }
        }

        
        for (const aobj of this.AObjects) {
          if (aobj.name == obj2name) {
            aobj.current = 1;
            aobj.locked = false;
            break;
          }
        }

        return;
      }
      if (this.act == "dump") {
        this.fsm.dump();
        return;
      }

      console.log("location: " + location);
      console.log("author: " + this.author);
      console.log("fsm" + this.fsm);
      console.log("initState: " + this.fsm.initState);
      this.fsm.transition(this.act);
      
      console.log("state: " + this.fsm.state);

      for (const loc of this.Locations) {
        if (loc.name == this.fsm.state) {

          this.loc.name = loc.name;
          this.loc.description = loc.description;
          this.loc.short = loc.short;
          this.loc.condition = loc.condition;

          console.log("location: " + loc.name + " condition: " + loc.condition);
          this.instruction = "";
          break;
        }
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.advent {
  padding:40px;
  width: 100%;
  min-height: 150px;
  background-color: white;
  border-bottom: 1px dotted #ddd;
}
.advent .block {
  color: #7b7b7b;
}
.advent .block h1 {
  font-family: "Roboto", sans-serif;
  font-weight: 100;
  font-size: 45px;
  line-height: 60px;
  letter-spacing: 10px;
  padding-bottom: 45px;
}
.advent .block p {
  font-size: 23px;
  line-height: 40px;
  font-family: "Roboto", sans-serif;
  font-weight: 300;
  letter-spacing: 3px;
}

.advent .block ul {
  padding: 0;
  margin: 0;
  list-style: none;
}

</style>
