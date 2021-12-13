<template>
  <div class="banner">
    <div class="alert alert-primary" v-if="show_bannerhint">
        <h6> {{bannerhint}}</h6>
    </div>
    <div class="alert alert-primary" v-if="show_banner">
      <p>{{banner}}</p>
      <p v-if="banner2 !=''">{{banner2}}</p>
      <p v-if="banner3 !=''">{{banner3}}</p>
    </div>
  </div>
</template>

<script>
export default {
  name: "Banner",
  props: {
    banner: {
      type: String
    },
    banner2: {
      type: String
    },
    banner3: {
      type: String
    },
    bannerhint: {
        type: String
    },
    show_bannerhint: {
      type: Boolean,
    },    
    show_banner: {
        type:Boolean 
    }
  },  
  watch:  {
      show_banner: function() {
          this.update_msg();
          this.$forceUpdate();
      },
      show_bannerhint: function() {
          this.update_msghint();
          this.$forceUpdate();
      }
  },
  methods: {
      update_msg: function() {
          console.log("banner: " + this.banner);
      },
      update_msghint: function () {
          console.log("banner hint:" + this.bannerhint);
      }
  }
};
</script>

<style>
.banner .block {
  color: #e3e3e4;
}
.banner .block h1 {
  font-family: "Roboto", sans-serif;
  font-weight: 100;
  font-size: 45px;
  line-height: 60px;
  letter-spacing: 10px;
  padding-bottom: 45px;
}
.banner .block p {
  font-size: 23px;
  line-height: 40px;
  font-family: "Roboto", sans-serif;
  font-weight: 300;
  letter-spacing: 3px;
}
</style>
