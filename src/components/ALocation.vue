<template>
  <div class="alocation">
    <div class="block" v-if="showDesc">
      <h1>{{ name }}</h1>
      <p>{{ description }}</p>
      <mark v-if="lighted==false"> dark </mark>
      <mark v-if="birdhint==true"> bird hint </mark>
    </div>
    <div class="block" v-else>
      <h1>{{ name }}</h1>
      <p>{{ short }}</p>
      <mark v-if="lighted==false"> dark </mark>
      <mark v-if="birdhint==true"> bird hint </mark>
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
      type: Number
    },
    description: {
      type: String,
    },
    short: {
      type: String,
    },
    lighted : {
      type: Boolean,
      default: function() { return false; },
    },

    liquid: {
      type: Boolean,
      default: function() { return false; }
    },
    birdhint: {
      type:Boolean,
      default: function() { return false; }
    },
    condition: {
      type: String,
    },
    showDesc: {
      type: Boolean,
    }
  },
  created: function() {
    this.showDesc = true;
  }
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
