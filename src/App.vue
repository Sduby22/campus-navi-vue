<template>
  <div class="fixed" style="width: 350px" v-show="isRoutingEmpty">
    <search-head
      @get-route="getRoute"
      @setbeg="setBeg"
      @setdest="setDest"
      @setpass="setPass"
      :selected="selected"
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
    @bike="getRoute('bike')"
    @other="getRoute('walk')"
    @fastest="fastest"
  />
  <div style="position: fixed; z-index: -1">
    <bupt-map @choose-point="choosePoint" :routing="routing" :start="naviing" @stop-navi="stopNavi"/>
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
      selected: {},
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
    async stopNavi(x, y, isshahe) {
      this.naviing = false;
      if (x&&y) {
        let obj = {
          coor: [x,y],
          isshahe: isshahe
        }
        this.clearroute()
        await this.choosePoint(obj)
      }
    },
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
    async pixeltoGeo(p) {
      let token = p.isshahe ? this.shahe : this.xitucheng
      let obj = {
        uuid: token,
        x: p.coor[0]+0.0001,
        y: p.coor[1]+0.0001
      }
      let data = (await axios.post(`${config.api}/geo/pos`, obj)).data
      return [data.data.lnt, data.data.lat]
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
    async choosePoint(p) {
      if (this.routing.data.length != 0) return
      console.log('用户手动选择起点');
      let geo = await this.pixeltoGeo(p)
      this.routing.beg = {
        geo,
        coor: p.coor,
        isshahe: p.isshahe
      };
      this.selected= Object.assign(this.routing.beg)
    },
    async setBeg(beg) {
      console.log(`用户输入起点: ${beg}`);
      this.routing.beg = {
        coor: await this.geotoPixel(buptPoints[beg]),
        geo: buptPoints[beg],
        isshahe: buptPoints[beg][1] > 40,
      };
    },
    async setDest(dest) {
      console.log(`用户输入终点: ${dest}`);
      this.routing.dest = {
        coor: await this.geotoPixel(buptPoints[dest]),
        geo: buptPoints[dest],
        isshahe: buptPoints[dest][1] > 40,
      };
    },
    async setPass(pass) {
      console.log(`用户输入途径点: ${pass}`);
      let coors = pass.map(x=>buptPoints[x])
      let x = await Promise.all(
        coors.map(async (x) => {
          return { geo: buptPoints[x], coor: await this.geotoPixel(x), isshahe: x[1] > 40 };
        })
      );
      this.routing.pass = x;
    },
    clearroute() {
      this.routing.data = [];
      this.routing.path = [];
    },
    async getRoute(str="walk") {
      console.log("寻路中...");
      let del = this.routing.data.length
      let p = this.routing.beg;
      this.routing.speed = str==="walk" ? config.walkspeed : config.bikespeed
      for (let x of this.routing.pass) {
        await this.getRouteOneStep(p, x, str);
        p = x;
      }
      await this.getRouteOneStep(p, this.routing.dest, str);
      this.routing.data.splice(0,del)
    },
    async getRouteOneStep(beg, end, str) {
      const isshahe = (obj) => {
        return obj.isshahe
      };
      const shaheRealy = {geo:buptPoints["沙河-北京邮电大学沙河校区-西门"], isshahe: true};
      const xituchengRelay = {geo:buptPoints["西土城-北京邮电大学西门"], isshahe: false};
      if (isshahe(beg) && !isshahe(end)) {
        await this.getRouteOneStep(beg, shaheRealy, str);
        await this.getRouteOneStep(xituchengRelay, end, str);
      } else if (!isshahe(beg) && isshahe(end)) {
        await this.getRouteOneStep(beg, xituchengRelay, str);
        await this.getRouteOneStep(shaheRealy, end, str);
      } else {
        let isxitucheng = !isshahe(beg) 
        let coor1 = beg.geo
        let coor2 = end.geo
        let res = (await this.getRouteApi(coor1, coor2, isxitucheng, str==="bike")).data.data;
        res["shahe"] = !isxitucheng;
        this.routing.data.push(res)
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
    getRouteApi(coor1, coor2, isxitucheng = false, bike=false) {
      let token = isxitucheng ? this.xitucheng : this.shahe;
      let obj = {
        uuid: token,
        lat1: coor1[1],
        lnt1: coor1[0],
        lat2: coor2[1],
        lnt2: coor2[0],
        speed: bike ? config.bikespeed : config.walkspeed,
      };
      if (bike) obj['vehicle']='bike'
      return axios.post(`${config.api}/nav/route`, obj);
    },
  },
  mounted() {
    this.apiInit()
    // setTimeout(() => this.apiInit(), 1000);
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
