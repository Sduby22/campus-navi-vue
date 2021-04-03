<template>
  <div :class="{'route-tabs-parent':true,'route-tabs-parent-collapsed':collapsed}">
    <el-tabs class="route-tabs" type="border-card" tab-position="top">
      <el-tab-pane v-for="item in panes" :key="item.name">
        <template #label>
          <span><i :class="item.icon"></i> {{ item.name }}</span>
        </template>
      </el-tab-pane>
    </el-tabs>
  <div class="close-btn">
    <el-button plain :icon="hideIcon" @click="hide()" size="small"></el-button>
  </div>
  </div>
</template>

<script>
export default {
  name: "show-route",
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
      collapsed: false,
      hideIcon: 'el-icon-arrow-left',
    };
  },
  computed: {},
  methods: {
    hide() {
      this.collapsed=!this.collapsed
      this.hideIcon=this.collapsed?'el-icon-arrow-right':'el-icon-arrow-left'
    }
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
  transform: translateX(-350px);
  transition-duration: 200ms;
}
.route-tabs {
}
.close-btn {
  position: absolute;
  right: -45px;
  top: 0px;
}
</style>
