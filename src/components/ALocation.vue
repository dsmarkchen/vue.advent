<template>
  <div class="alocation">
    <div class="block" v-if="showDesc">
      <h1>{{ name }}</h1>
      <p>{{ description }}</p>
    </div>
    <div class="block" v-else>
      <h1>{{ name }}</h1>
      <p>{{ short }}</p>
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
    description: {
      type: String,
    },
    short: {
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
</style>
