<template>
  <div class="score">
    <div class="alert alert-primary" v-if="showscore">
      <p>Your score: {{ score }}</p>
      <p>Max score: {{max_score}}</p>

      <p>{{msg}}</p>
    </div>
  </div>
</template>

<script>
export default {
  name: "Score",
  props: {
    score: {
      type: Number
    },
    showscore: {
      type: Boolean,
    },    
  },
  created: function() {
    this.showscore = false;
  },
  watch:  {
      showscore: function() {
          this.update_msg();
      }
  },
  data: function() {
      return {
          max_score:  350,
          msg: ""
      }
  },  
  methods: {
      update_msg: function() {
        if(this.score < 35) {
            this.msg = "You are obviously a rank amateur.  Better luck next time.";
        }
        else if(this.score < 100) {
            this.msg = "Your score qualifies you as a novice class adventurer.";
        }else if(this.score < 130) {
            this.msg = "You have achieved the rating \"Experienced Adventurer\".";
        }else if(this.score < 200) {
            this.msg = "You may now consider yourself a \"Seasoned Adventurer\".";
        }else if(this.score < 250) {
            this.msg = "You have reached \"Junior Master\" status.";
        }else if(this.score < 300) {
            this.msg = "Your score puts you in Master Adventurer Class C.";
        }else if(this.score < 330) {
            this.msg = "Your score puts you in Master Adventurer Class B.";
        }else if(this.score < 349) {
            this.msg = "Your score puts you in Master Adventurer Class A.";
        }else if(this.score < 9999) {
            this.msg = "All of Adventuredom gives tribute to you, Adventure Grandmaster!";
        }


      }
  }
};
</script>

<style>
.score .block {
  color: #e3e3e4;
}
.score .block h1 {
  font-family: "Roboto", sans-serif;
  font-weight: 100;
  font-size: 45px;
  line-height: 60px;
  letter-spacing: 10px;
  padding-bottom: 45px;
}
.score .block p {
  font-size: 23px;
  line-height: 40px;
  font-family: "Roboto", sans-serif;
  font-weight: 300;
  letter-spacing: 3px;
}
</style>
