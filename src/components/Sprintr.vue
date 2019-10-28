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
    <br/>
    <br/>
    <span style="font-size:xx-small">Made with love by Ace of the <a href="https://facebook.com/WordsAfterDark">Words After Dark</a> crew.</span>
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
      // Break timer
      // Recalculate time on every tick since setInterval is inaccurate
      var now = new Date();
      this.timeTillSprint = Math.floor((this.sprintStartTime - now) / 1000);
      if (this.timeTillSprint < 1)  
      {
        // Timer is up
        this.timeTillSprint = 0;
        clearInterval(this.timer);

        // Play start sound
        if (now < this.sprintEndTime && (this.playSound === "true")) {
          var playPromise = this.startAudio.play();
          if (playPromise !== null) {
            playPromise.catch(() => {
              this.startAudio.play();
            });
          }
        }

        // Start sprint timer
        this.sprintTime = Math.floor((this.sprintEndTime - now) / 1000);
        this.sprintTimer = setInterval(() => this.sprintCountDown(), 500);
      }
    },
    sprintCountDown: function() {
      // Timer for sprint itself
      // Recalculate time on every tick since setInterval is inaccurate
      var now = new Date();
      this.sprintTime = Math.floor((this.sprintEndTime - now) / 1000);
      if (this.sprintTime < 1) 
      {
        // Timer is up
        this.sprintTime = 0;
        clearInterval(this.sprintTimer);

        // Play end sound
        if (now < this.sprintEndTime && (this.playSound === "true")) {
          var playPromise = this.endAudio.play();
          if (playPromise !== null) {
            playPromise.catch(() => {
              this.endAudio.play();
            });
          }
        }

        // Calculate next sprint
        this.sprintStartTime = new Date(
          this.sprintEndTime.getTime() + this.breakDuration * 60000
        );
        this.sprintEndTime = new Date(
          this.sprintStartTime.getTime() + this.Duration * 60000
        );

        // Start break period and break timer
        now = new Date();
        this.timeTillSprint = Math.floor((this.sprintStartTime - now) / 1000);
        this.timer = setInterval(() => this.countdownToSprint(), 500);
      }
    },
    padTime: function(time) {
      return (time < 10 ? "0" : "") + time;
    },
    getSprint: function() {
      // Create sprint url from sprint builder
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
    // Grab parameters from URL
    let uri = window.location.search.substring(1);
    let params = new URLSearchParams(uri);
    this.Duration = params.get("duration");

    // If URL is empty, show builder
    if (this.Duration === null) {
      this.noSprint = 1;

      // Set up example data
      var now = new Date();
      var exampleDate = new Date(
        now.getTime() + 15 * 60000
      );
      this.desiredHour = exampleDate.getHours();
      this.desiredMinute = Math.ceil(exampleDate.getMinutes() / 10) * 10;
      this.desiredDuration = 15;
      this.desiredBreak = 10;
    }
    else 
    {
    // Otherwise, initialize more stuff
    var hour = params.get("hour");
    var minute = params.get("minute");
    this.breakDuration = params.get("break");
    this.playSound = params.get("sound");

    // Set up alert sounds
    this.startAudio = new Audio(
      "http://soundbible.com/mp3/Zen%20Temple%20Bell-SoundBible.com-2070036999.mp3"
    );
    this.startAudio.volume = 0.2;
    this.endAudio = new Audio(
      "http://soundbible.com/mp3/Metal_Gong-Dianakc-109711828.mp3"
    );
    this.endAudio.volume = 0.2;
    }

    // Calculate sprint start and end times for timers
    var now = new Date();
    this.sprintStartTime = new Date();
    this.sprintStartTime.setHours(hour, minute, 0);
    this.sprintEndTime = new Date(
      this.sprintStartTime.getTime() + this.Duration * 60000
    );
    this.timeTillSprint = Math.floor((this.sprintStartTime - now) / 1000);
    
    // Start break timer
    this.timer = setInterval(() => this.countdownToSprint(), 1000);
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
