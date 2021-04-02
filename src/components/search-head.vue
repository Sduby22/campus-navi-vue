<template>
  <div v-show="!route" class="search-input">
    <el-input
      class="el-input"
      placeholder="请输入地点"
      v-model="dest.value"
      @focus="currentInput = dest"
      clearable
      suffix-icon="el-icon-search"
    >
      <template #append>
        <el-button
          type="primary"
          @click="invRoute()"
          icon="el-icon-s-promotion"
        ></el-button>
      </template>
    </el-input>
  </div>
  <el-card
    class="main-card"
    shadow="hover"
    :body-style="searchCard"
    v-show="currentInput.value && !route"
  >
    <div class="main-card-div">
      <li
        v-for="(x, index) in stringQuery"
        :key="index"
        @click="changeInput(x)"
      >
        <i class="el-icon-location-outline" />{{ x }}
      </li>
    </div>
  </el-card>

  <el-card
    class="route-card"
    shadow="always"
    :body-style="{ padding: '20px', overflowY: 'auto' }"
    v-show="route"
  >
    <template #header>
      <div class="close-btn">
        <i class="el-icon-close" @click="invRoute()"></i>
      </div>
      <div class="left-font">路径规划</div>
      <el-input
        class="el-input mgbtm mgtop"
        placeholder="起点"
        v-model="begin.value"
        @focus="currentInput = begin"
        clearable
        suffix-icon="el-icon-search"
      />
      <el-button
        class="mgbtm"
        plain
        round
        size="small"
        @click="passby.push({ value: '' })"
      >
        + 途径点</el-button
      >
      <el-button
        class="mgbtm"
        plain
        round
        size="small"
        @click="passby.length && passby.length--"
      >
        - 途径点</el-button
      >
      <el-input
        v-for="(x, index) in passby"
        :key="index"
        class="mgbtm el-input"
        :placeholder="'途径点' + (index + 1)"
        v-model="passby[index].value"
        @focus="currentInput = passby[index]"
        clearable
        suffix-icon="el-icon-top-right"
      >
        <template #append>
          <el-button
            icon="el-icon-minus"
            @click="passby.splice(index, 1)"
          ></el-button>
        </template>
      </el-input>
      <el-input
        class="el-input mgbtm"
        placeholder="终点"
        v-model="dest.value"
        @focus="currentInput = dest"
        clearable
        suffix-icon="el-icon-search"
      />
    </template>
    <li v-for="(x, index) in stringQuery" :key="index" @click="changeInput(x)">
      <i class="el-icon-location-outline" />{{ x }}
    </li>
  </el-card>
</template>

<script>
export default {
  name: "search-head",
  setup() {
    const searchCard = {
      fontSize: "16px",
      fontColor: "grey",
      padding: "5px",
    };
    return {
      searchCard,
    };
  },
  data() {
    return {
      currentInput: {},
      begin: {
        value: "",
      },
      dest: {
        value: "",
      },
      passby: [],
      route: false,
    };
  },
  computed: {
    stringQuery() {
      if (!this.currentInput.value) return null;
      var arr = [];
      for (let i = 0; i != 99; i++) arr.push("asd");
      return arr;
      // return [this.currentInput.value, "aaa"];
    },
  },
  methods: {
    invRoute() {
      this.route = !this.route;
      this.currentInput = {};
    },
    changeInput(str) {
      this.currentInput.value = str;
    },
  },
};
</script>

<style lang="scss" scoped>
.left-font {
  margin-left: 5px;
  text-align: left;
}
.el-input {
  :deep(input) {
    color: black;
  }
}
.search-input {
  max-width: 350px;
}
.mgbtm {
  margin-bottom: 5px;
}
.mgtop {
  margin-top: 20px;
}
li {
  padding: 5px 0;
  list-style-type: none;
}
.main-card {
  margin-top: 10px;
  width: 350px;
}
.main-card-div {
  max-height: 80vh;
  overflow-y: auto;
}
.route-card {
  :deep(.el-card) {
    display: flex;
    flex-direction: column;
    max-height: 95vh;
  }
  position: relative;
  width: 350px;
}
.card-input {
  width: 100%;
}
.close-btn {
  position: absolute;
  right: 20px;
}
</style>
