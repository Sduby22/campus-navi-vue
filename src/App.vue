<template>
  <div v-show="!routing">
    <search-head @get-route="getRoute"></search-head>
  </div>
  <show-route :routing="routing" @clear-route="routing=null"/>
</template>

<script>
import SearchHead from './components/search-head.vue';
import showRoute from './components/show-route'

export default {
  name: "App",
  components: {
    'show-route': showRoute,
     SearchHead,
  },
  data() {
    return {
      routing: null,
    }
  },
  methods: {
    getRoute() {
      this.routing = {}
    }
  }
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
