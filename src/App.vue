<template>
  <div class="fixed" style="width: 350px" v-show="isRoutingEmpty">
    <search-head
      @get-route="getRoute"
      @setbeg="setBeg"
      @setdest="setDest"
      @setpass="setPass"
    ></search-head>
  </div>
  <show-route
    class="fixed"
    v-if="!isRoutingEmpty"
    :routing="routing"
    @start-navi="naviing = true"
    @end-navi="naviing = false"
    @clear-route="clearroute"
    @shortest="shortest"
    @fastest="fastest"
  />
  <div style="position: fixed; z-index: -1">
    <bupt-map :routing="routing" :start="naviing" />
  </div>
</template>

<script>
import BuptMap from "./components/bupt-map.vue";
import SearchHead from "./components/search-head.vue";
import buptPoints from "./assets/buptPoints.json";
import showRoute from "./components/show-route";
import axios from "axios";
import config from "./config.json";

export default {
  name: "App",
  components: {
    "show-route": showRoute,
    SearchHead,
    BuptMap,
  },
  data() {
    return {
      naviing: false,
      null_routing: {
        data: [],
        beg: null,
        dest: null,
        pass: [],
        path: [],
      },
      routing: {
        data: [],
        beg: null,
        dest: null,
        pass: [],
        path: [],
      },
      xitucheng: "",
      shahe: "",
    };
  },
  computed: {
    isRoutingEmpty() {
      return this.routing.data.length === 0;
    },
  },
  methods: {
    shortest() {
      this.routing.path.length = 0;
      for (let x of this.routing.data) {
        this.routing.path.push({ shahe: x.shahe, path: x.shortest.path });
      }
    },
    fastest() {
      this.routing.path.length = 0;
      for (let x of this.routing.data) {
        this.routing.path.push({ shahe: x.shahe, path: x.fastest.path });
      }
    },
    geoisShahe(coor) {
      return coor[1] > 40;
    },
    async geotoPixel(coor) {
      let token = this.geoisShahe(coor) ? this.shahe : this.xitucheng;
      let obj = {
        uuid: token,
        lat: coor[1],
        lnt: coor[0],
      };
      let data = (await axios.post(`${config.api}/geo/pixel`, obj)).data;
      return [data.data.x, data.data.y];
    },
    async setBeg(beg) {
      this.routing.beg = {
        coor: await this.geotoPixel(buptPoints[beg]),
        isshahe: buptPoints[beg][1] > 40,
      };
    },
    async setDest(dest) {
      this.routing.dest = {
        coor: await this.geotoPixel(buptPoints[dest]),
        isshahe: buptPoints[dest][1] > 40,
      };
    },
    async setPass(pass) {
      let coors = pass.map(x=>buptPoints[x])
      let x = await Promise.all(
        coors.map(async (x) => {
          return { coor: await this.geotoPixel(x), isshahe: x[1] > 40 };
        })
      );
      this.routing.pass = x;
    },
    clearroute() {
      this.routing.data = [];
      this.routing.path = [];
    },
    async getRoute(begin, pass, dest) {
      let p = begin;
      for (let x of pass) {
        await this.getRouteOneStep(p, x);
        p = x;
      }
      await this.getRouteOneStep(p, dest);
    },
    async getRouteOneStep(beg, end) {
      const isshahe = (str) => {
        return str.slice(0, 2) === "沙河";
      };
      const shaheRealy = "沙河-北京邮电大学沙河校区-西门";
      const xituchengRelay = "西土城-北京邮电大学西门";
      if (isshahe(beg) && !isshahe(end)) {
        await this.getRouteOneStep(beg, shaheRealy);
        await this.getRouteOneStep(xituchengRelay, end);
      } else if (!isshahe(beg) && isshahe(end)) {
        await this.getRouteOneStep(beg, xituchengRelay);
        await this.getRouteOneStep(shaheRealy, end);
      } else {
        let isxitucheng = beg.slice(0, 2) === "西土";
        let coor1 = buptPoints[beg];
        let coor2 = buptPoints[end];
        let res = (await this.getRouteApi(coor1, coor2, isxitucheng)).data.data;
        res["shahe"] = !isxitucheng;
        this.routing.data.push(res);
      }
      return Promise.resolve()
    },
    async apiInit() {
      let obj1 = {
        map: "bupt-xitucheng",
        minLat: 39.95628900323458,
        minLnt: 116.3487629866333,
        maxLat: 39.96370515558461,
        maxLnt: 116.35526320361328,
        coefficient: 2.9187342,
      };
      let obj2 = {
        map: "bupt-shahe",
        minLat: 40.15351,
        minLnt: 116.27679,
        maxLat: 40.15974,
        maxLnt: 116.28934,
        coefficient: 2.3955597,
      };
      this.xitucheng = (
        await axios.post(`${config.api}/nav/load`, obj1)
      ).data.data.instanceToken;
      this.shahe = (
        await axios.post(`${config.api}/nav/load`, obj2)
      ).data.data.instanceToken;
    },
    getRouteApi(coor1, coor2, isxitucheng = false) {
      let token = isxitucheng ? this.xitucheng : this.shahe;
      let obj = {
        uuid: token,
        lat1: coor1[1],
        lnt1: coor1[0],
        lat2: coor2[1],
        lnt2: coor2[0],
        speed: 10.0001,
      };
      return axios.post(`${config.api}/nav/route`, obj);
    },
  },
  mounted() {
    setTimeout(() => this.apiInit(), 1000);
  },
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
