<template>
  <div style="width: 1px; height: 1px">
    <div
      :class="{
        'route-tabs-parent': true,
        'route-tabs-parent-collapsed': collapsed,
      }"
      v-if="!navi"
    >
      <el-tabs v-model="active" class="route-tabs" type="border-card" tab-position="top" @tab-click="handleclick">
        <el-tab-pane v-if="!bus" name="walk">
          <template #label>
            <span><i class="el-icon-user"></i> 步行</span>
          </template>
          <div
            class="pane"
          >
            <el-card shadow="hover" class="tab-card" @click="shortest">
              <template #header>
                <div class="card-header">
                  <span>最短路径</span>
                </div>
              </template>
              <div class="text">用时: {{ secondsToHms(timeShortest) }}</div>
              <div class="text">距离: {{ distanceShortest }}m</div>
            </el-card>
            <el-card shadow="hover" class="tab-card" @click="fastest">
              <template #header>
                <div class="card-header">
                  <span>最短时间</span>
                </div>
              </template>
              <div class="text">用时: {{ secondsToHms(timeFastest) }}</div>
              <div class="text">距离: {{ distanceFastest }}m</div>
            </el-card>
          </div>
          <div style="text-align:center">
            <el-button @click="startnavi" type="primary" :disabled="routing.path.length===0" style="margin:20px 0 0 0">开始导航</el-button>
          </div>
        </el-tab-pane>
        <el-tab-pane v-if="!bus" name="bike">
          <template #label>
            <span><i class="el-icon-bicycle"></i> 骑行</span>
          </template>
          <div
            class="pane"
          >
            <el-card shadow="hover" class="tab-card" @click="shortest">
              <template #header>
                <div class="card-header">
                  <span>最短路径</span>
                </div>
              </template>
              <div class="text">用时: {{ secondsToHms(timeShortest) }}</div>
              <div class="text">距离: {{ distanceShortest }}m</div>
            </el-card>
            <el-card shadow="hover" class="tab-card" @click="fastest">
              <template #header>
                <div class="card-header">
                  <span>最短时间</span>
                </div>
              </template>
              <div class="text">用时: {{ secondsToHms(timeFastest) }}</div>
              <div class="text">距离: {{ distanceFastest }}m</div>
            </el-card>
          </div>
          <div style="text-align:center">
            <el-button @click="startnavi" type="primary" :disabled="routing.path.length===0" style="margin:20px 0 0 0">开始导航</el-button>
          </div>
        </el-tab-pane>
        <el-tab-pane v-if="bus" name="bus">
          <template #label>
            <span><i class="el-icon-truck"></i> 公交</span>
          </template>
          <div
            class="pane"
          >
            <el-card shadow="hover" class="tab-card" @click="shortest">
              <template #header>
                <div class="card-header">
                  <span>最短路径</span>
                </div>
              </template>
              <div class="text">行走用时: {{ secondsToHms(timeShortest) }}</div>
              <div class="text">行走距离: {{ distanceShortest }}m</div>
            </el-card>
            <el-card shadow="hover" class="tab-card" @click="fastest">
              <template #header>
                <div class="card-header">
                  <span>最短时间</span>
                </div>
              </template>
              <div class="text">行走用时: {{ secondsToHms(timeFastest) }}</div>
              <div class="text">行走距离: {{ distanceFastest }}m</div>
            </el-card>
          </div>
            <div class="bus-schedule">
              <p>校车时刻表: 9:00 11:00 13:00 15:00 17:00</p>
            </div>
            <div class="bus-schedule">
              <p>公交车每15分钟发车一辆</p>
            </div>
          <div style="text-align:center">
            <el-button @click="startnavi" type="primary" :disabled="routing.path.length===0" style="margin:20px 0 0 0">开始导航</el-button>
          </div>
        </el-tab-pane>
      </el-tabs>
      <div :class="collapsed ? 'collapse-btn-collapsed' : 'collapse-btn'">
        <el-button
          style="border-radius: 4px 0 0 4px"
          plain
          icon="el-icon-close"
          @click="close()"
          size="small"
        ></el-button>
        <el-button
          style="margin-left: 0"
          class="btn-right-radius"
          plain
          :icon="hideIcon"
          @click="hide()"
          size="small"
        ></el-button>
      </div>
    </div>
    <div v-if="navi">
      <el-button type="danger" class="stop-navi" @click="stopnavi" round>停止导航</el-button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    routing: Object,
  },
  name: "show-route",
  emits: ["clear-route", "shortest", "fastest", "start-navi", "end-navi", "bike", "other"],
  setup() {
    const panes = [
      {
        name: "步行",
        icon: "el-icon-user",
      },
      {
        name: "骑车",
        icon: "el-icon-bicycle",
      },
      {
        name: "公交",
        icon: "el-icon-truck",
      },
    ];
    return {
      panes,
    };
  },
  data() {
    return {
      active: "walk",
      navi: false,
      collapsed: false,
      hideIcon: "el-icon-arrow-left",
    };
  },
  computed: {
    bus() {
      let list = this.routing.data.map(x=>x.shahe)
      let change = 0;
      list.reduce((old,cur)=>{ 
        if(old===cur) {
          return old;
        } else {
          change+=1
          return cur
        } 
      })
      return change
    },
    timeShortest() {
      return this.routing.data.reduce(
        (old, cur) => Math.floor(old + cur["shortest"].time),
        0
      )+this.bus*3600;
    },
    distanceShortest() {
      return this.routing.data.reduce(
        (old, cur) => Math.floor(old + cur["shortest"].distance),
        0
      );
    },
    timeFastest() {
      return this.routing.data.reduce(
        (old, cur) => Math.floor(old + cur["fastest"].time),
        0
      )+this.bus*3600;
    },
    distanceFastest() {
      return this.routing.data.reduce(
        (old, cur) => Math.floor(old + cur["fastest"].distance),
        0
      );
    },
  },
  methods: {
    secondsToHms(d) {
      d = Number(d);
      var h = Math.floor(d / 3600);
      var m = Math.floor(d % 3600 / 60);
      var s = Math.floor(d % 3600 % 60);

      var hDisplay = h > 0 ? h + "小时" : "";
      var mDisplay = m > 0 ? m + "分" : "";
      var sDisplay = s > 0 ? s + "秒" : "";
      return hDisplay + mDisplay + sDisplay; 
    },
    handleclick() {
      if (this.active === "bike")
        this.$emit('bike')
      else
        this.$emit('other')
    },
    startnavi() {
      this.$emit('start-navi')
      this.navi=true
    },
    stopnavi() {
      this.$emit('end-navi')
      this.navi=false
    },
    shortest() {
      this.$emit("shortest");
    },
    fastest() {
      this.$emit("fastest");
    },
    hide() {
      this.collapsed = !this.collapsed;
      this.hideIcon = this.collapsed
        ? "el-icon-arrow-right"
        : "el-icon-arrow-left";
    },
    close() {
      this.$emit("clear-route");
    },
  },
};
</script>

<style lang="scss" scoped>
.pane {
  display: flex; 
  justify-content: space-between;
  align-items: center;
}
.route-tabs-parent {
  width: 350px;
  position: relative;
  text-align: left;
  transition-duration: 200ms;
}
.route-tabs-parent-collapsed {
  transform: translateX(-400px);
  transition-duration: 200ms;
}

.collapse-btn-collapsed {
  position: absolute;
  right: -137px;
  top: 4px;
  transition-duration: 200ms;
}

.btn-right-radius {
  margin-left: 0px;
  border-radius: 0 4px 4px 0;
}

.collapse-btn {
  position: absolute;
  right: 5px;
  top: 4px;
  transition-duration: 200ms;
}

.tab-card {
  text-align: left;
  width: 150px;
}

.text {
  color: #6f6f6f;
  font-size: 14px;
}

.stop-navi {
  position: fixed;
  bottom: 20px;
  left: 50px;
}

.bus-schedule > p {
  font-size: 14px;
  margin-bottom:0;
  margin-top:5px;
}
</style>
