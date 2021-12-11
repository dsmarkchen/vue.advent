<template>
  <div class="aobject">
    <div class="block">
      <h6 v-show="showTitle">{{ name }} : {{ desc }}</h6>
      <p v-show="showTaken">taken: {{ taken }}</p>
      <ul>
        <div v-for="(note, index) in notes" v-bind:key="note">
          <li v-if="index == current && (taken != true)">            
              <div v-if="(taken != undefined) && (taken == false)">{{note}} </div>
              <div v-else>
                <div v-if="locked != undefined">{{note}} <mark> locked: {{ locked }} </mark> </div>                
                <div v-else>
                  <div v-if="taken == undefined && locked == undefined">{{ note }} </div>
                </div>
              </div>
          </li>
        </div>
      </ul>
      <p>{{ location }} </p>
    </div>
  </div>
</template>

<script>
export default {
  name: "AObject",
  props: {
    name: {
      type: String,
    },
    desc: {
      type: String,
    },
    notes: {
      type: Array,
    },
    location: {
      type: String,
    },
    taken: {
      type: Boolean,
    },
    using: {
      type: Boolean,
    },
    locked: {
      type: Boolean,
    },
    current: {
      type: Number,
    },
  },
  data: function () {
    return {    
      showTitle: false,
      showTaken: false,
    };
  },
};
</script>

<style>
.aobject .block {
  color: #7b7b7b;
}
.aobject .block h1,
h2,
h3,
h4,
h5,
h6 {
  font-family: "Roboto", sans-serif;
}
.aobject .block p {
  padding-left: 20px;
  font-size: 15px;
  line-height: 28px;
}
</style>
