<template>
  <div v-show="routing">
  <div
    :class="{
      'route-tabs-parent': true,
      'route-tabs-parent-collapsed': collapsed,
    }"
  >
    <el-tabs class="route-tabs" type="border-card" tab-position="top">
      <el-tab-pane v-for="item in panes" :key="item.name">
        <template #label>
          <span><i :class="item.icon"></i> {{ item.name }}</span>
        </template>
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
  emits: ['clear-route'],
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
      hideIcon: "el-icon-arrow-left",
    };
  },
  computed: {},
  methods: {
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
</style>
