<template>
  <div class="alocation" v-show="show_location">
    <div class="alert">
      <h6 v-if="showName">{{ name }}</h6>
      <p v-if="showDesc">{{ description }}</p>
      <p v-else>{{ short }}</p>
    </div>
  </div>
</template>

<script>
export default {
  name: "ALocation",
  props: {
    name: {
      type: String,
    },
    q: {
      type: Number,
    },
    description: {
      type: String,
    },
    short: {
      type: String,
    },
    lighted: {
      type: Boolean,
      default: function () {
        return false;
      },
    },
    oil: {
      type: Boolean,
      default: function () {
        return false;
      },
    },
    liquid: {
      type: Boolean,
      default: function () {
        return false;
      },
    },

    condition: {
      type: String,
    },
    show_location: {
      type: Boolean,
    },
    showDesc: {
      type: Boolean,
    },
  },
  watch: {
    show_location: function () {
      console.log("show_location:" + this.show_location);

      this.$forceUpdate();
    },
  },
  created: function () {
    this.showDesc = true;
    this.showName = false;
    this.show_location = false;
  },
  data: function () {
    return {
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
};
</script>

<style>
#alocation .block {
  color: #e3e3e4;
}
#alocation .block h1 {
  font-family: "Roboto", sans-serif;
  font-weight: 100;
  font-size: 45px;
  line-height: 60px;
  letter-spacing: 10px;
  padding-bottom: 45px;
}
#alocation .block p {
  font-size: 23px;
  line-height: 40px;
  font-family: "Roboto", sans-serif;
  font-weight: 300;
  letter-spacing: 3px;
}

.alocation .block ul {
  padding: 0;
  margin: 0;
  list-style: none;
}
</style>
