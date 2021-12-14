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
        ></Banner>
      </div>

        <div class="alert alert-primary" v-if="helpHint==true">  
          <p>{{helpMsg}}</p> 
          <p>{{helpMsg2}}</p> 
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
          :cavehint="loc.cavehint"
          :birdhint="loc.birdhint"
          :snakehint="loc.snakehint"
          :twisthint="loc.twisthint"
          :darkhint="loc.darkhint"
          :witthint="loc.witthint"
          :isDark="is_dark"
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
import Banner from "./Banner.vue"
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
    console.log("fsm transition: " + action + " current state: " + currentState) ;
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
    console.log("mfsm transition: " + action + " current state: " + currentState) ;
    for (var key of Object.keys(p)) {
      if (key == currentState) {
        if(!((Object.keys(p[key])).includes(action))) {
          context["_error"](action);          
          return;
        }
        for (var child_key of Object.keys(p[key])) {
          if (child_key == action) {
            console.log("do: " + p[key][child_key].action + " " + p[key][child_key].arguments);
            if(p[key][child_key].action != null) {
              if(p[key][child_key].arguments == undefined) {
                context[p[key][child_key].action]();
              }
              else {
                context[p[key][child_key].action](p[key][child_key].arguments);
              }
            }
            this.state = p[key][child_key].next_state;
            console.log("mfsm key:" + key + "child_key:" + child_key)
            console.log("mfsm next state: " +this.state)
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
      showscore: false,
      showObject: false,
      is_dark: false,
      fsm: new FSM("", []),
      author: new Person("", ""),
      act: "yes",
      loc: null,
      AObjects: [],
      Actions: ["in", "out"],
      Locations: locations,
      helpHint:false,
      show_location: false,

      pitchDarkMsg: "It is now pitch dark.  If you proceed you will most likely fall into a pit.",

      caveHintMsg:"The grate is very solid and has a hardened steel lock.  You cannot enter without a key, and there are no keys in sight.  I would recommend looking elsewhere for the keys.",
      birdHintMsg: "Something seems to be frightening the bird just now and you cannot catch it no matter what you try.  Perhaps you might try later.",
      snakeHintMsg:"You can't kill the snake, or drive it away, or avoid it, or anything like that.  There is a way to get by, but you don't have the necessary resources right now",
      darkHintMsg:"You can make the passages look less alike by dropping things.",
      wittHintMsg:"There is a way to explore that region without having to worry about falling into a pit.  None of the objects available is immediately useful for discovering the secret."

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
    var motionMap = {
        'welcome_hint' : {  
          'yes': {next_state: "idle", action: "_banner", arguments: "welcome"},
          'no':  {next_state: "idle", action: "_banner", arguments:"off"}
        },
        'idle' : {
          'info': {next_state: "idle", action: "_banner", arguments:"info"},
          'score': {next_state:"idle", action: "_score"},
          'help': {next_state: "idle", action: "_banner", arguments: "help"},
          'look': {next_state: "idle", action: "_look"},
          
          'in': {next_state: "idle", action: "_move", arguments: "in"},
          'out': {next_state: "idle", action: "_move", arguments: "out"},
          'north': {next_state: "idle", action: "_move", arguments: "north"},
          'south': {next_state: "idle", action: "_move", arguments: "south"},
          'west': {next_state: "idle", action: "_move", arguments: "west"},
          'east': {next_state: "idle", action: "_move", arguments: "east"},
          'up': {next_state: "idle", action: "_move", arguments: "up"},
          'down': {next_state: "idle", action: "_move", arguments: "down"},

          'xyzzy': {next_state: "idle", action: "_move", arguments: "xyzzy"},


          "take": {next_state: "idle", action: "_take", arguments: "take"},
          "drop": {next_state: "idle", action: "_take", arguments: "drop"},
          "on": {next_state: "idle", action: "_on"},
          "off": {next_state: "idle", action: "_off"},

          'dark': {next_state: "dark", action: "_banner", arguments: "dark"}
        },
        'dark' : {
          'on': {next_state: "idle", action: "on"},
          'look': {next_state: "dark", action: "_banner", arguments:"dark"}
        }

    };
    
    this.mFSM = new FSMEx(this, 'welcome_hint', motionMap);
    this.fsm = new FSM("road", transitions);
    this.loc = JSON.parse(JSON.stringify(this.Locations[0]));
    this.show_bannerhint = true;
    this.show_banner = false;
    this.helpHint = false;

    this.banner_hint_default = "Welcome to Adventure! Would you like instructions?";
    this.banner_msg_default = "Somewhere nearby is Colossal Cave, where others have found fortunes in treasure and gold, though it is rumored that some who enter are never seen again.  Magic is said to work in the cave.  I will be your eyes and hands.  Direct me with commands of one or two words.  I should warn you that I look at only the first five letters of each word, so you'll have to enter \"NORTHEAST\" as \"NE\" to distinguish it from \"NORTH\".  Should you get stuck, type \"HELP\" for some general hints. For information on how to end your adventure, etc., type \"INFO\".";
    this.banner_msg_default2 ="The first adventure program was developed by Willie Crowther. Most of the features of the current program were added by Don Woods; all of its bugs were added by Don Knuth."

    this.helpMsg="I know of places, actions, and things.  Most of my vocabulary describes places and is used to move you there.  To move, try words like forest, building, downstream, enter, east, west, north, south, up, or down.  I know about a few special objects, like a black rod hidden in the cave.  These objects can be manipulated using some of the action words that I know.  Usually you will need to give both the object and action words (in either order), but sometimes I can infer the object from the verb alone.  Some objects also imply verbs; in particular, \"inventory\" implies \"take inventory\", which causes me to give you a list of what you're carrying.  The objects have side effects; for instance, the rod scares the bird.  Usually people having trouble moving just need to try a few more words.  Usually people trying unsuccessfully to manipulate an object are attempting something beyond their (or my!) capabilities and should try a completely different tack.  To speed the game you can sometimes move long distances with a single word.  For example, \"building\" usually gets you to the building from anywhere above ground except when lost in the forest.  Also, note that cave passages turn a lot, and that leaving a room to the north does not guarantee entering the next from the south.";
    this.helpMsg2 = "Good luck!";
    this.infoMsg ="If you want to end your adventure early, say \"quit\".  To get full credit for a treasure, you must have left it safely in the building, though you get partial credit just for locating it.  You lose points for getting killed, or for quitting, though the former costs you more. There are also points based on how much (if any) of the cave you've managed to explore; in particular, there is a large bonus just for getting in (to distinguish the beginners from the rest of the pack), and there are other ways to determine whether you've been through some of the more harrowing sections.  If you think you've found all the treasures, just keep exploring for a while.  If nothing interesting happens, you haven't found them all yet.  If something interesting DOES happen, it means you're getting a bonus and have an opportunity to garner many more points in the master's section. I may occasionally offer hints if you seem to be having trouble. If I do, I'll warn you in advance how much it will affect your score to accept the hints.  Finally, to save paper, you may specify \"brief\", which tells me never to repeat the full description of a place unless you explicitly ask me to.";
    this.welcome_hint = true;
    this.welecome();

    this.help_hint = false;
  },
  methods: {
    _error(warningMsg) {
      console.log("fsm error: " + warningMsg);
    },
    _score() {
      console.log("fsm score: "  );
      this.showscore = true;
    },
    _banner(str) {
      console.log("banner " + str    );
      if(str.startsWith("off")) {
        this.show_bannerhint = false;
        this.show_banner = false;
        this.instruction = "ok";
        return;
      }
      if(str.startsWith("dark")) {
        this.show_bannerhint = false;
        this.show_banner = true;
        this.banner_msg =  this.pitchDarkMsg; 
        this.banner_msg2 = "";
        this.banner_msg3 = "";
      }
      if(str.startsWith("welcome")) {
        this.show_bannerhint = false;
        this.show_banner = true;
        this.banner_hint = this.banner_hint_default;
        this.banner_msg = this.banner_msg_default;
        this.banner_msg2 = this.banner_msg_default2;
        this.banner_msg3 = this.banner_msg3_default3;
      }
      if(str.startsWith("help")) {
        this.show_bannerhint = false;
        this.show_banner = true;
        this.banner_msg = this.helpMsg;
        this.banner_msg2 = this.helpMsg2;
        this.banner_msg3 = "";
      }
      if(str.startsWith("info")) {
        this.show_bannerhint = false;
        this.show_banner = true;
        this.banner_msg = this.infoMsg;
        this.banner_msg2 = "";
        this.banner_msg3 = "";
      }
    },

    _look() {
      this.show_bannerhint = false;
      this.show_banner = false;
      this.banner_msg = "";
      this.show_location = true;
    },

    _move(act) {
      console.log("move: " + act);   
      this.show_location = true;   
      this.fsm.transition(act);

      for (const loc of this.Locations) {
        if (loc.name == this.fsm.state) {
          this.loc = Object.assign({}, loc);
          
          if (this.loc.lighted == true) {
            this.is_dark = false;
          } 
          else {
            if(this.isLampOn()) {
              this.is_dark = false;
            }
            else {
              //this.is_dark = true;
              this.mFSM.transition("dark");
              
              
            }
          }
          console.log("is_dark: " + this.is_dark);
          console.log("location: " + loc.name + ", lighted: " + loc.lighted);
          this.instruction = "";
          break;
        }
      }
    },
        
    _take(act) {
      console.log("take or drop: " + act);      
      if (this.act.startsWith("take")) {
        var objname = act.slice(4).trim();
        console.log("take: " + objname);
        for (const aobj of this.AObjects) {
          if (aobj.name == objname) {
            aobj.taken = true;
            this.instruction = objname + "is taken";
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
            if(aobj.taken) {
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
          if(aobj.taken) {
            aobj.using = false;
            this.instruction = offname + " is off";
          }
          break;
        }
      }
        
      
    },

    welecome() {
      if(this.welcome_hint) {
        this.show_bannerhint = true;
        this.show_banner = false;
        this.banner_hint = this.banner_hint_default;
        this.banner_msg = this.banner_msg_default;
        this.banner_msg2 = this.banner_msg_default2;
        this.banner_msg3 = this.banner_msg3_default3;
      }
      else {
        this.show_bannerhint = false;
        this.show_banner = true;    
      }
    },
    help() {
      if(this.help_hint) {
        this.show_bannerhint = true;
        this.show_banner = false;
        this.banner_hint = "Need help?"
        this.banner_msg = this.helpMsg;
        this.banner_msg2 = this.helpMsg2;
        this.banner_msg3 = "";
      }
      else {
        this.show_bannerhint = false;
        this.show_banner = true;
        this.banner_msg = this.helpMsg;
        this.banner_msg2 = this.helpMsg2;
        this.banner_msg3 = "";
        
      }
    },
    convent(action) {
      let act = action;
      if(action.length == 1) {
          if(action == "n") {
            act = "north";
          }
          if(action == "s") {
            act = "south";
          }

          if(action == "e") {
            act = "east";
          }
                    
          if(action == "w") {
            act = "east";
          }


      }
      if(action == "key") {
        action = "keys";
      }
      return act;

    },
    isLampOn() {
      let onname = "lamp";
      for (const aobj of this.AObjects) {
        if (aobj.name == onname) {
          console.log("lamp: taken: " + aobj.taken + ",using:" + aobj.using);
          if(aobj.using == undefined || aobj.taken == false) {
            return false;
          }
          else if (aobj.using) {
            return true; // taken and use
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
      this.mFSM.transition(this.act);
      console.log("next mFSM.state: " + this.mFSM.state);
      
      this.$forceUpdate();
        
      if(this.show_location == false)  return;

      if (act.startsWith("on")) {
        var onname = act.slice(2).trim();
        console.log("use (on): " + onname);
        for (const aobj of this.AObjects) {
          if (aobj.name == onname) {
            if(aobj.taken)
              aobj.using = true;
            break;
          }
        }        
      }

      if (act.startsWith("off")) {
        var offname = this.act.slice(3).trim();
        console.log("use (off): " + onname);
        for (const aobj of this.AObjects) {
          if (aobj.name == offname) {
            if(aobj.taken)
              aobj.using = false;
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
      if (act == "dump") {
        this.fsm.dump();
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
