<template>
  <div v-show="navi">
  <div
    :class="{
      'route-tabs-parent': true,
      'route-tabs-parent-collapsed': collapsed,
    }"
  >
    <el-tabs class="route-tabs" type="border-card" tab-position="top">
      <el-tab-pane v-if="!isbus">
        <template #label>
          <span><i class="el-icon-user"></i> 步行</span>
        </template>
        <div class="pane" style="display:flex;justify-content: space-between">
        <el-card shadow="hover" class="tab-card" @click="shortest">
          <template #header>
            <div class="card-header">
              <span>最短路径</span>
            </div>
          </template>
          <div class="text">用时: {{ timeShortest }}s</div>
          <div class="text">距离: {{ distanceShortest }}m</div>
        </el-card>
        <el-card shadow="hover" class="tab-card" @click="fastest">
          <template #header>
            <div class="card-header">
              <span>最短时间</span>
            </div>
          </template>
          <div class="text">用时: {{ timeFastest }}s</div>
          <div class="text">距离: {{ distanceFastest }}m</div>
        </el-card>
        </div>
      </el-tab-pane>
      <el-tab-pane v-if="!isbus">
        <template #label>
          <span><i class="el-icon-bicycle"></i> 骑行</span>
        </template>
        <div class="pane" style="display:flex;justify-content: space-between">
        <el-card shadow="hover" class="tab-card" @click="shortest">
          <template #header>
            <div class="card-header">
              <span>最短路径</span>
            </div>
          </template>
          <div class="text">用时: {{ timeShortest }}s</div>
          <div class="text">距离: {{ distanceShortest }}m</div>
        </el-card>
        <el-card shadow="hover" class="tab-card" @click="fastest">
          <template #header>
            <div class="card-header">
              <span>最短时间</span>
            </div>
          </template>
          <div class="text">用时: {{ timeFastest }}s</div>
          <div class="text">距离: {{ distanceFastest }}m</div>
        </el-card>
        </div>
      </el-tab-pane>
      <el-tab-pane v-if="isbus">
        <template #label>
          <span><i class="el-icon-truck"></i> 公交</span>
        </template>
        <div class="pane" style="display:flex;justify-content: space-between">
        <el-card shadow="hover" class="tab-card" @click="shortest">
          <template #header>
            <div class="card-header">
              <span>最短路径</span>
            </div>
          </template>
          <div class="text">用时: {{ timeShortest }}s</div>
          <div class="text">距离: {{ distanceShortest }}m</div>
        </el-card>
        <el-card shadow="hover" class="tab-card" @click="fastest">
          <template #header>
            <div class="card-header">
              <span>最短时间</span>
            </div>
          </template>
          <div class="text">用时: {{ timeFastest }}s</div>
          <div class="text">距离: {{ distanceFastest }}m</div>
        </el-card>
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
  </div>
</template>

<script>
export default {
  props: {
    routing: Object,
  },
  name: "show-route",
  emits: ['clear-route', 'shortest', 'fastest'],
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
      navi: false,
      collapsed: false,
      hideIcon: "el-icon-arrow-left",
    };
  },
  computed: {
    isbus() {
      return false
    },
    timeShortest() {
      return this.routing.data.reduce((old, cur) => Math.floor(old + cur["shortest"].time), 0)
    },
    distanceShortest() {
      return this.routing.data.reduce((old, cur) => Math.floor(old + cur["shortest"].distance), 0)
    },
    timeFastest() {
      return this.routing.data.reduce((old, cur) => Math.floor(old + cur["fastest"].time), 0)
    },
    distanceFastest() {
      return this.routing.data.reduce((old, cur) => Math.floor(old + cur["fastest"].distance), 0)
    }
  },
  methods: {
    shortest() {
      this.$emit('shortest')
    },
    fastest() {
      this.$emit('fastest')
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
</style>
