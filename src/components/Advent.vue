<template>
  <div class="container">
    <div class="advent">
      <div class="block">
        <Banner
          :bannerhint="banner_hint"
          :banner="banner_msg"
          :banner2="banner_msg2"
          :banner3="banner_msg3"
          :show_banner="show_banner"
          :show_bannerhint="show_bannerhint"
          :is_danger="is_danger"
        ></Banner>
      </div>

      <div class="block">
        <ALocation
          :show_location="show_location"
          :name="loc.name"
          :description="loc.description"
          :short="loc.short"
          :showDesc="full"
          :lighted="loc.lighted"
          :liquid="loc.liquid"          
        ></ALocation>

        <ul>
          <div v-for="item in AObjects" :key="item.name">
            <li v-if="item.location == loc.name && item.locked != undefined">
              <AObject
                :name="item.name"
                :desc="item.desc"
                :notes="item.notes"
                :taken="item.taken"
                :using="item.using"
                :locked="item.locked"
                :current="item.current"
              ></AObject>
            </li>

            <li v-if="item.location == loc.name && item.taken != undefined">
              <AObject
                :name="item.name"
                :desc="item.desc"
                :notes="item.notes"
                :taken="item.taken"
                :using="item.using"
                :locked="item.locked"
                :current="item.current"
              ></AObject>
            </li>
          </div>
        </ul>

        <Score :score="score" :show_score="show_score"> </Score>

        <div v-show="show_input">
          <b-form-group
            :description="instruction"
            label=""
            label-for="input-1"
            valid-feedback=""
            :invalid-feedback="input_invalid"
            :state="input_state"
          >
            <b-form-input
              id="input-1"
              v-model="act"
              v-on:keyup.enter="next"
              trim
            ></b-form-input>
          </b-form-group>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import ALocation from "./ALocation.vue";
import Score from "./Score.vue";
import Banner from "./Banner.vue";
import AObject from "./AObject.vue";
import objects from "../json/objects.json";
import locations from "../json/locations.json";
import location_transitions from "../json/location_transitions.json";
import motion_transitions from "../json/motion_transitions.json";
import vocabulary from "../json/vocabulary.json";


function FSM(initState, transitions) {
  this.initState = initState;
  this.transitions = transitions;
  this.state = initState;
  this.transition = function (action) {
    var currentState = this.state;
    var p = this.transitions;
    console.log(
      "fsm transition: " + action + " current state: " + currentState
    );
    for (var key of Object.keys(p)) {
      if (key == currentState) {
        for (var child_key of Object.keys(p[key])) {
          if (child_key == action) {
            this.state = p[key][child_key];
            console.log("fsm next_state: " + this.state);
            break;
          }
        }
      }
    }
  };
}

function FSMEx(context, initState, transitions) {
  this.context = context;
  this.initState = initState;
  this.transitions = transitions;
  this.state = initState;
  this.transition = function (actions) {
    let words = actions.split(" ");
    let action = words[0];
    var currentState = this.state;
    var p = this.transitions;
    console.log(
      "mfsm transition: " + action + " current state: " + currentState
    );
    for (var key of Object.keys(p)) {
      if (key == currentState) {
        if (!Object.keys(p[key]).includes(action)) {
          context["_error"](action);
          return;
        }
        for (var child_key of Object.keys(p[key])) {
          if (child_key == action) {
            console.log(
              "do: " +
                p[key][child_key].action +
                " " +
                p[key][child_key].arguments
            );
            if (p[key][child_key].action != null) {
              if (p[key][child_key].arguments == undefined) {
                context[p[key][child_key].action]();
              } else {
                context[p[key][child_key].action](p[key][child_key].arguments);
              }
            }
            this.state = p[key][child_key].next_state;
            console.log("mfsm key:" + key + ",child_key:" + child_key);
            console.log("mfsm next state: " + this.state);
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
    Banner,
  },
  data: function () {
    return {
      instruction: "Let us know what you wanna do",
      score: 0,
      is_danger: false,
      show_score: false,
      showObject: false,
      is_dark: false,
      fsm: new FSM("", []),      
      act: "yes",
      loc: null,
      AObjects: [],
      Actions: ["in", "out"],
      Locations: locations,

      show_location: false,

      pitchDarkMsg:
        "It is now pitch dark.  If you proceed you will most likely fall into a pit.",

      caveHintMsg:
        "The grate is very solid and has a hardened steel lock.  You cannot enter without a key, and there are no keys in sight.  I would recommend looking elsewhere for the keys.",
      birdHintMsg:
        "Something seems to be frightening the bird just now and you cannot catch it no matter what you try.  Perhaps you might try later.",
      snakeHintMsg:
        "You can't kill the snake, or drive it away, or avoid it, or anything like that.  There is a way to get by, but you don't have the necessary resources right now",
      darkHintMsg:
        "You can make the passages look less alike by dropping things.",
      wittHintMsg:
        "There is a way to explore that region without having to worry about falling into a pit.  None of the objects available is immediately useful for discovering the secret.",
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
    this.show_input = true;
    this.showObject = false;
    this.score = 20;
    this.full = true;    
    this.AObjects = objects;
    
    this.mFSM = new FSMEx(this, "begin", motion_transitions);
    this.lfsm = new FSM("road", location_transitions);
    this.loc = JSON.parse(JSON.stringify(this.Locations[0]));
    this.show_bannerhint = true;
    this.show_banner = false;

    this.giveup_msg_hint = "Do you really want to quit now?";
    this.banner_hint_default =
      "Welcome to Adventure! Would you like instructions?";
    this.banner_msg_default =
      'Somewhere nearby is Colossal Cave, where others have found fortunes in treasure and gold, though it is rumored that some who enter are never seen again.  Magic is said to work in the cave.  I will be your eyes and hands.  Direct me with commands of one or two words.  I should warn you that I look at only the first five letters of each word, so you\'ll have to enter "NORTHEAST" as "NE" to distinguish it from "NORTH".  Should you get stuck, type "HELP" for some general hints. For information on how to end your adventure, etc., type "INFO".';
    this.banner_msg_default2 =
      "The first adventure program was developed by Willie Crowther. Most of the features of the current program were added by Don Woods; all of its bugs were added by Don Knuth.";

    this.helpMsg =
      'I know of places, actions, and things.  Most of my vocabulary describes places and is used to move you there.  To move, try words like forest, building, downstream, enter, east, west, north, south, up, or down.  I know about a few special objects, like a black rod hidden in the cave.  These objects can be manipulated using some of the action words that I know.  Usually you will need to give both the object and action words (in either order), but sometimes I can infer the object from the verb alone.  Some objects also imply verbs; in particular, "inventory" implies "take inventory", which causes me to give you a list of what you\'re carrying.  The objects have side effects; for instance, the rod scares the bird.  Usually people having trouble moving just need to try a few more words.  Usually people trying unsuccessfully to manipulate an object are attempting something beyond their (or my!) capabilities and should try a completely different tack.  To speed the game you can sometimes move long distances with a single word.  For example, "building" usually gets you to the building from anywhere above ground except when lost in the forest.  Also, note that cave passages turn a lot, and that leaving a room to the north does not guarantee entering the next from the south.';
    this.helpMsg2 = "Good luck!";
    this.infoMsg =
      "If you want to end your adventure early, say \"quit\".  To get full credit for a treasure, you must have left it safely in the building, though you get partial credit just for locating it.  You lose points for getting killed, or for quitting, though the former costs you more. There are also points based on how much (if any) of the cave you've managed to explore; in particular, there is a large bonus just for getting in (to distinguish the beginners from the rest of the pack), and there are other ways to determine whether you've been through some of the more harrowing sections.  If you think you've found all the treasures, just keep exploring for a while.  If nothing interesting happens, you haven't found them all yet.  If something interesting DOES happen, it means you're getting a bonus and have an opportunity to garner many more points in the master's section. I may occasionally offer hints if you seem to be having trouble. If I do, I'll warn you in advance how much it will affect your score to accept the hints.  Finally, to save paper, you may specify \"brief\", which tells me never to repeat the full description of a place unless you explicitly ask me to.";

    this.mFSM.transition("yes");
  },
  methods: {
    _error(warningMsg) {
      console.log("fsm error: " + warningMsg);
    },
    _end() {
      console.log("bye bye");
      this.show_core = false;

      this.show_bannerhint = false;
      this.show_banner = false;
      this.show_location = false;
      this.show_input = false;
    },
    _score() {
      console.log("fsm score: ");
      this.show_score = true;
    },
    _banner2(obj) {
      console.log("banner2 obj:" + obj);
      if (Object.hasOwn(obj, "is_danger")) {
        this.is_danger = obj.is_danger;
        console.log("is_danger:" + obj.is_danger)
      }

      if (Object.hasOwn(obj, "instruction")) {
        this.instruction = obj.instruction;
        console.log("instruction:" + obj.instruction)
      }

      if (Object.hasOwn(obj, "show_location")) {
        this.show_location = obj.show_location;
      }

      if (Object.hasOwn(obj, "show_score")) {
        this.show_score = obj.show_score;
      }

      if (Object.hasOwn(obj, "show_bannerhint")) {
        this.show_bannerhint = obj.show_bannerhint;
      }
      if (Object.hasOwn(obj, "banner_hint")) {
        this.banner_hint = obj.banner_hint;
      }

      if (Object.hasOwn(obj, "show_banner")) {
        this.show_banner = obj.show_banner;
      }
      if (Object.hasOwn(obj, "banner_msg")) {
        this.banner_msg = obj.banner_msg;
      }
      if (Object.hasOwn(obj, "banner_msg2")) {
        this.banner_msg2 = obj.banner_msg2;
      }
      if (Object.hasOwn(obj, "banner_msg3")) {
        this.banner_msg3 = obj.banner_msg3;
      }
    },
    _banner(str) {
      console.log("banner " + str);
      if (str.startsWith("off")) {
        this.show_bannerhint = false;
        this.show_banner = false;
        this.instruction = "ok";
        return;
      }
      if (str.startsWith("dark")) {
        this.show_bannerhint = false;
        this.show_banner = true;
        this.banner_msg = this.pitchDarkMsg;
        this.banner_msg2 = "";
        this.banner_msg3 = "";
      }
      if (str.startsWith("welcome")) {
        this.show_bannerhint = false;
        this.show_banner = true;
        this.banner_hint = this.banner_hint_default;
        this.banner_msg = this.banner_msg_default;
        this.banner_msg2 = this.banner_msg_default2;
        this.banner_msg3 = this.banner_msg3_default3;
      }
      if (str.startsWith("help")) {
        this.show_bannerhint = false;
        this.show_banner = true;
        this.banner_msg = this.helpMsg;
        this.banner_msg2 = this.helpMsg2;
        this.banner_msg3 = "";
      }
      if (str.startsWith("info")) {
        this.show_bannerhint = false;
        this.show_banner = true;
        this.banner_msg = this.infoMsg;
        this.banner_msg2 = "";
        this.banner_msg3 = "";
      }
    },

    _move(act) {
      console.log("move: " + act);
      this.show_location = true;
      this.show_bannerhint = false;
      this.show_banner = false;
      this.banner_msg = "";
      this.banner_msg2 = "";
      this.banner_msg3 = "";
      this.instruction = "";

      this.lfsm.transition(act);
    },

    _take() {
      
      if (this.act.startsWith("take")) {
        var objname = this.act.slice(4).trim();
        console.log("take: " + objname);
        for (const aobj of this.AObjects) {
          if (aobj.name == objname) {
            if(aobj.taken == true) {              
              this.instruction = objname + " is already taken.";
            }
            else {
              aobj.taken = true;
              this.instruction = objname + " is taken.";
              
            }
            break;
          }
        }
        return;
      }
    },
    _drop() {
      
      if (this.act.startsWith("drop")) {
        var objname = this.act.slice(4).trim();
        console.log("drop: " + objname);
        for (const aobj of this.AObjects) {
          if (aobj.name == objname) {
            if(aobj.taken == true) {
              this.instruction = objname + " is dropped."              
              aobj.take = false;
            }
            else {
              
              this.instruction = objname + " is not available.";
              
            }
            break;
          }
        }
        return;
      }
    },


    _on() {
      var act = this.act;
      var onname = act.slice(2).trim();
      console.log("use (on): " + onname);
      for (const aobj of this.AObjects) {
        if (aobj.name == onname) {
          if (aobj.taken) {
            aobj.using = true;
            this.instruction = onname + " is on";
          }
          break;
        }
      }
    },
    _off() {
      var act = this.act;
      var offname = act.slice(3).trim();
      console.log("use (off): " + offname);
      for (const aobj of this.AObjects) {
        if (aobj.name == offname) {
          if (aobj.taken) {
            aobj.using = false;
            this.instruction = offname + " is off";
          }
          break;
        }
      }
    },

    
    convent(action) {
      let act = action.toLowerCase();
      if (action.length == 1) {
        if (action == "n") {
          act = "north";
        }
        if (action == "s") {
          act = "south";
        }

        if (action == "e") {
          act = "east";
        }

        if (action == "w") {
          act = "east";
        }
      }
      if (action == "key") {
        action = "keys";
      }
      return act;
    },
    isLampOn() {
      let onname = "lamp";
      for (const aobj of this.AObjects) {
        if (aobj.name == onname) {
          console.log("lamp: taken: " + aobj.taken + ",using:" + aobj.using);
          if (aobj.using == undefined || aobj.taken == false) {
            return false;
          } else if (aobj.using) {
            return true; // taken and use
          } else {
            return false;
          }
        }
      }
      return false;
    },

    isAObjTaken(onname) {      
      for (const aobj of this.AObjects) {
        if (aobj.name == onname) {
          console.log(onname + ": taken: " + aobj.taken + ",using:" + aobj.using);
          if (aobj.taken) {
            return true;
          } else {
            return false;
          }
        }
      }
      return false;
    },
    isAObjTakenAndUsing(onname) {      
      for (const aobj of this.AObjects) {
        if (aobj.name == onname) {
          if ((aobj.taken == true) && (aobj.using == true)) {
            return true;
          } else {
            return false;
          }
        }
      }
      return false;
    },


    next() {
      let act = this.convent(this.act);
      console.log("act:" + act);
      console.log("start mFSM.state: " + this.mFSM.state);

      if (this.mFSM.state == "welcome_hint" && act != "yes" && act != "no") {
        act = "not_yes_no";
      }

      this.mFSM.transition(act);
      console.log("next mFSM.state: " + this.mFSM.state);

      if(this.mFSM.state == "dark" && ((act == "south") || (act == "north"))) {
        this.mFSM.transition("move_in_darkness");
      }

      for (const loc of this.Locations) {
        if (loc.name == this.lfsm.state) {
          this.loc = Object.assign({}, loc);
          if(loc.cavehint == true) {
            this.mFSM.transition("cave_hint");
          }
          if(loc.birdhint == true) {
            if (this.isAObjTaken("rod")) {          
              this.mFSM.transition("bird_hint");
            }
          }
          if(loc.snakehint == true) {
            this.mFSM.transition("snake_hint");
          }
          if(loc.darkhint == true) {
            this.mFSM.transition("dark_hint");
          }
          if(loc.witthint == true) {
            this.mFSM.transition("witt_hint");
          }

          if (loc.lighted == false) {
          
            if (this.isAObjTakenAndUsing("lamp")) {          
              console.log("lamp_on")    ;
              if(this.mFSM.state == "dark")
                this.mFSM.transition("lamp_on");
            } else {
              console.log("dark_no_light")    ;
              if(this.mFSM.state == "idle")
                this.mFSM.transition("dark_no_light");
            }
          }          
          break;
        }
      }
      
      

      this.$forceUpdate();

      if (this.show_location == false) return;

      if (act.startsWith("on")) {
        var onname = act.slice(2).trim();
        console.log("use (on): " + onname);
        for (const aobj of this.AObjects) {
          if (aobj.name == onname) {
            if (aobj.taken) aobj.using = true;
            break;
          }
        }
      }

      if (act.startsWith("off")) {
        var offname = this.act.slice(3).trim();
        console.log("use (off): " + onname);
        for (const aobj of this.AObjects) {
          if (aobj.name == offname) {
            if (aobj.taken) aobj.using = false;
            break;
          }
        }
      }

      if (this.act.startsWith("take")) {
        var objname = act.slice(4).trim();
        console.log("take: " + objname);
        for (const aobj of this.AObjects) {
          if (aobj.name == objname) {
            aobj.taken = true;
            break;
          }
        }
        return;
      }
      if (this.act.startsWith("drop")) {
        var dropname = act.slice(4).trim();
        console.log("drop: " + dropname);
        for (const aobj of this.AObjects) {
          if (aobj.name == dropname) {
            aobj.taken = false;
            break;
          }
        }
        return;
      }

      if (act.startsWith("open")) {
        var obj2name = act.slice(4).trim();
        console.log("open: " + obj2name);

        // check keys
        if (obj2name == "grate") {
          for (const aobj of this.AObjects) {
            if (aobj.name == "keys") {
              if (aobj.taken == false) {
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
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.advent {
  padding: 40px;
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
