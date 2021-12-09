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
            <li v-if="item.location == loc.name && item.taken == false">
              <AObject :name="item.name" :description="item.desc" :taken="item.taken"></AObject>
            </li>
          </div>
        </ul>

        <Score :score="score" :showscore="showscore"> </Score>

        <b-form-input v-model="act"></b-form-input>

        <b-button @click="next"> enter </b-button>
      </div>
    </div>
  </div>
</template>

<script>
import ALocation from "./ALocation.vue";
import Score from "./Score.vue";
import AObject from "./AObject.vue";
import locations from "../json/locations.json";
import transitions from "../json/transitions.json";

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
      score: 0,
      showscore: false,
      showObject: false,

      fsm: new FSM("", []),
      author: new Person("", ""),
      act: "in",
      loc: null,
      AObjects: [],
      objsInPlace: [],
      Actions: ["in", "out"],
      Locations: locations,
    };
  },
  created: function () {
    this.showObject = false;
    this.score = 20;
    this.full = true;
    this.author = new Person("", "");
    this.AObjects = [
      {
        name: "lamp",
        desc: "lanturn",
        location: "house",
        taken: false,
      },
      {
        name: "keys",
        desc: "",
        location: "house",
        taken: false,
      },
      {
        name: "bottle",
        desc: "a bottle of water",
        location: "house",
        taken: false,
      },
      {
        name: "cage",
        desc: "Wicker cage",
        location: "cobbles",
        taken: false,
      },
    ];

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
      if(this.act.startsWith("take")) {
        var objname = this.act.slice(4).trim();
        console.log("take: " + objname);
        for (const aobj of this.AObjects) {
        if (aobj.name == objname) {
          aobj.taken = true;
          break;
        }
      }
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
          break;
        }
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#advent .block {
  color: #e3e3e4;
}
#advent .block h1 {
  font-family: "Roboto", sans-serif;
  font-weight: 100;
  font-size: 45px;
  line-height: 60px;
  letter-spacing: 10px;
  padding-bottom: 45px;
}
#advent .block p {
  font-size: 23px;
  line-height: 40px;
  font-family: "Roboto", sans-serif;
  font-weight: 300;
  letter-spacing: 3px;
}

.advent .block ul {
  padding-top: 35px;
  line-height: 27px;
/*  margin-left: 25px; */
  list-style-type: none;

}
.advent .block ul li {
	text-indent: -1.4em;
    color: #7B7B7B;
}

</style>
