<template>
  <Launches />
</template>

<script>
import Launches from "./components/Launches.vue";

export default {
  name: "App",
  components: {
    Launches
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #343d46;
}

body {
  background-color: #343d46;
}
</style>
