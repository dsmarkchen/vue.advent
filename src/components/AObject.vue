<template>
  <div class="aobject">
    <div class="block" >
      <h6>Object: {{ name }}</h6>

      <p>{{ description }}</p>
      <p> taken: {{taken}} <p>
      <p>{{ location }}</p>
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
    description: {
      type: String,
    },
    location: {
      type: String,
    },
    taken: {
      type: Boolean,
    },

  }
};
</script>

<style>
.aobject .block {
  color: #7b7b7b; 
}
.aobject .block h1,h2,h3,h4,h5,h6 {
  font-family: "Roboto", sans-serif;
}
.aobject .block p {  
  font-size: 15px;
  line-height: 28px;
}
</style>
