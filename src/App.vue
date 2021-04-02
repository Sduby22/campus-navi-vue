<template>
  <searchhead/>
</template>

<script>
import searchhead from './components/search-head'

export default {
  name: "App",
  components: {
    searchhead,
  },
};
</script>

<style>
searchhead {
  float: left;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 20px;
}
</style>
