<template>
  <div id="app">
    <h1>Welcome to the Words After Dark Word Sprint Tool!</h1>
    <a href="https://words-after-dark.square.site"><img width="25%" src="./assets/logo.png"></a>
    <Sprintr/>
  </div>
</template>

<script>
import Sprintr from "./components/Sprintr";

export default {
  name: "App",
  components: {
    Sprintr
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
