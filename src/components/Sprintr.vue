<template>
  <div>
    <template v-if="timeTillSprint > 0">
      <h3>Break time! The next {{Duration}} minute sprint starts in:</h3>
      <h4>{{ minutesTillSprint }}:{{ secondsTillSprint }}</h4>
    </template>
    <template v-else-if="sprintTime > 0">
      <h3>Let the words flow!</h3>
      <h1>{{ sprintMinutes }}:{{ sprintSeconds }}</h1>
    </template>
    <template v-else-if="noSprint === 1">
      <h3>Create a sprint!</h3>
      <div>I want a sprint that starts at <input v-model="desiredHour" type=number min=0 max=23 style="width:35px"></input>:<input v-model="desiredMinute" type=number min=0 max=60 style="width:35px"></input></div>
      <div>And has a duration of <input v-model="desiredDuration" type=number min=1 style="width:35px"></input> minutes.</div>
      <div>After the sprint, there will be a <input v-model="desiredBreak" type=number min=1 style="width:35px"></input> minute break after which the sprint will repeat.</div>
      <div>Enable start/stop sounds? <input v-model="desiredSound" type=checkbox></input></div>
      <button v-on:click="getSprint">Go!</button>
    </template>
    <template v-else>
      <h3>Loading...</h3>
    </template>
  </div>
</template>

<script>
export default {
  name: "Sprintr",
  data: function() {
    return {
      sprintStartTime: null,
      sprintEndTime: null,
      timer: null,
      sprintTimer: null,
      timeTillSprint: 0,
      sprintTime: 0,
      breakDuration: 0,
      Duration: 0,
      startAudio: null,
      endAudio: null,
      playSound: 0,
      noSprint: 0,
      desiredHour: 0,
      desiredMinute: 0,
      desiredDuration: 0,
      desiredBreak: 0,
      desiredSound: false,
      customUrl: null
    };
  },
  methods: {
    countdownToSprint: function() {
      if (this.timeTillSprint >= 1) {
        this.timeTillSprint--;
      } else {
        this.timeTillSprint = 0;
        clearInterval(this.timer);
        var now = new Date();
        if (now < this.sprintEndTime && (this.playSound === "true")) {
          var playPromise = this.startAudio.play();
          if (playPromise !== null) {
            playPromise.catch(() => {
              this.startAudio.play();
            });
          }
        }
        this.sprintTime = Math.floor((this.sprintEndTime - now) / 1000);
        this.sprintTimer = setInterval(() => this.sprintCountDown(), 1000);
      }
    },
    sprintCountDown: function() {
      if (this.sprintTime >= 1) {
        this.sprintTime--;
      } else {
        this.sprintTime = 0;
        clearInterval(this.sprintTimer);
        //calculate next sprint
        this.sprintStartTime = new Date(
          this.sprintEndTime.getTime() + this.breakDuration * 60000
        );
        this.sprintEndTime = new Date(
          this.sprintStartTime.getTime() + this.Duration * 60000
        );
        var now = new Date();
        if (now < this.sprintEndTime && (this.playSound === "true")) {
          var playPromise = this.endAudio.play();
          if (playPromise !== null) {
            playPromise.catch(() => {
              this.endAudio.play();
            });
          }
        }
        this.timeTillSprint = Math.floor((this.sprintStartTime - now) / 1000);
        this.timer = setInterval(() => this.countdownToSprint(), 1000);
      }
    },
    padTime: function(time) {
      return (time < 10 ? "0" : "") + time;
    },
    getSprint: function() {
      window.location.href = "https://word-sprint.net?hour=" + this.desiredHour + "&minute=" + this.desiredMinute + "&duration=" + this.desiredDuration + "&break=" + this.desiredBreak + "&sound=" + this.desiredSound; 
    }
  },
  computed: {
    minutesTillSprint: function() {
      return this.padTime(Math.floor(this.timeTillSprint / 60));
    },
    secondsTillSprint: function() {
      return this.padTime(Math.floor(this.timeTillSprint % 60));
    },
    sprintMinutes: function() {
      return this.padTime(Math.floor(this.sprintTime / 60));
    },
    sprintSeconds: function() {
      return this.padTime(Math.floor(this.sprintTime % 60));
    }
  },
  created: function() {
    let uri = window.location.search.substring(1);
    let params = new URLSearchParams(uri);
    var hour = params.get("hour");
    var minute = params.get("minute");
    this.Duration = params.get("duration");
    if (this.Duration === null) {
      this.noSprint = 1;
    }
    else {

    
    this.breakDuration = params.get("break");
    this.playSound = params.get("sound");
    var now = new Date();
    this.sprintStartTime = new Date();
    this.sprintStartTime.setHours(hour, minute, 0);
    this.sprintEndTime = new Date(
      this.sprintStartTime.getTime() + this.Duration * 60000
    );
    this.timeTillSprint = Math.floor((this.sprintStartTime - now) / 1000);
    this.timer = setInterval(() => this.countdownToSprint(), 1000);
    this.startAudio = new Audio(
      "http://soundbible.com/mp3/Zen%20Temple%20Bell-SoundBible.com-2070036999.mp3"
    );
    this.startAudio.volume = 0.2;
    this.endAudio = new Audio(
      "http://soundbible.com/mp3/Metal_Gong-Dianakc-109711828.mp3"
    );
    this.endAudio.volume = 0.2;
    }
  }
};
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
