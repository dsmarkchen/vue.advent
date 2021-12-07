<template>
  <div class="advent">
    <h1>{{ msg }}</h1>
    <p> {{info}}</p>
    <b-form-input v-model="act"></b-form-input>
    
    <b-button @click="update()" v-model="act" >Enter</b-button>
  </div>
</template>

<script>
export default {
  name: 'Advent',
  data() {
    return {
        msg: "Welcome to Adventure!!",
        info: "You are standing at the end of a road before a small brick building.",
        act: "Need help?"    
    };
  },
  methods: {
    update() {

        let userObj = {
          name: "Sammy",
          email: "sammy@example.com",
          plan: "Pro"
        };

        let userStr = JSON.stringify(userObj);

        console.log(userStr);
        this.info = "tada";
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
