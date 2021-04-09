<template>
  <div class="fixed" style="width:350px" v-show="!routing">
    <search-head @get-route="getRoute"></search-head>
  </div>
  <show-route class="fixed" :routing="routing" @clear-route="routing=null"/>
  <div style="position:fixed; z-index: -1">
  <bupt-map/>
  </div>
</template>

<script>
import BuptMap from './components/bupt-map.vue';
import SearchHead from './components/search-head.vue';
import showRoute from './components/show-route'

export default {
  name: "App",
  components: {
    'show-route': showRoute,
    SearchHead,
    BuptMap,
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
.fixed {
  top: 20px;
  left: 20px;
  position: fixed;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
