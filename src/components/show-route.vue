<template>
  <el-tabs class="route-tabs" type="border-card" tab-position="top">
    <el-tab-pane v-for="item in panes" :key="item.name">
      <template #label>
        <span><i :class="item.icon"></i> {{ item.name }}</span>
      </template>
    </el-tab-pane>
  </el-tabs>
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
    return {};
  },
  computed: {},
  methods: {},
};
</script>

<style lang="scss" scoped>
.route-tabs {
  width: 350px;
}
</style>
