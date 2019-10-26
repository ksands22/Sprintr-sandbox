<template>
  <div>{{ minutes }}:{{ seconds }}</div>
</template>

<script>
export default {
  name: "HelloWorld",
  data: function() {
    return {
      timer: null,
      startHour: 0,
      startMinute: 0,
      duration: 0,
      timeTillSprint: 100
    };
  },
  methods: {
    countdown: function() {
      if (this.timeTillSprint >= 1) {
        this.timeTillSprint--;
      } else {
        this.timeTillSprint = 0;
        this.timeTillSprint = 100;
      }
    },
    padTime: function(time) {
      return (time < 10 ? '0' : '') + time;
    }
    },
    computed: {
      minutes: function() {
        return this.padTime(Math.floor(this.timeTillSprint / 60));
      },
      seconds: function() {
        return this.padTime(Math.floor(this.timeTillSprint % 60));
      }
    },
  created: function() {
    this.timer = setInterval(() => this.countdown(), 1000);
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
