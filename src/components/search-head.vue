<template>
  <!-- Main Card -->
  <div v-show="!route" class="search-input">
    <el-input
      class="el-input"
      placeholder="请输入地点"
      v-model="dest.value"
      @focus="currentInput = dest"
      @input="checkInput"
      clearable
      suffix-icon="el-icon-search"
    >
      <!-- Route Button -->
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
    v-show="cardShow && !route"
  >
    <div class="main-card-div">
      <li
        v-for="(x, index) in stringQuery"
        :key="index"
        @click="changeInput(x.value);route=true"
        :style="x.style"
      >
        <i class="el-icon-location-outline"/>{{ x.name }}
      </li>
    </div>
  </el-card>

  <!-- Route Card -->
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
        @input="checkInput"
        clearable
        suffix-icon="el-icon-search"
      />
      <el-button
        class="mgbtm"
        plain
        round
        size="small"
        @click="
          passby.push({ value: '', type: 'pass' });
          setpoints('pass');
        "
      >
        + 途径点</el-button
      >
      <el-button
        class="mgbtm"
        plain
        round
        size="small"
        @click="
          passby.length && passby.length--;
          setpoints('pass');
        "
      >
        - 途径点</el-button
      >
      <div class="passby-scroll">
        <el-input
          v-for="(x, index) in passby"
          :key="index"
          class="mgbtm el-input"
          :placeholder="'途径点' + (index + 1)"
          v-model="passby[index].value"
          @focus="currentInput = passby[index]"
          @input="checkInput"
          clearable
          suffix-icon="el-icon-top-right"
        >
          <template #append>
            <el-button
              icon="el-icon-minus"
              @click="
                passby.splice(index, 1);
                setpoints('pass');
              "
            ></el-button>
          </template>
        </el-input>
      </div>
      <el-input
        class="el-input mgbtm"
        placeholder="终点"
        v-model="dest.value"
        @focus="currentInput = dest"
        @input="checkInput"
        clearable
        suffix-icon="el-icon-search"
      />
      <el-button type="primary" :disabled="!checkAllLegal" @click="getRoute"
        >规划路线</el-button
      >
    </template>
      <li
        v-for="(x, index) in stringQuery"
        :key="index"
        @click="changeInput(x.value);route=true"
        :style="x.style"
      >
        <i class="el-icon-location-outline"/>{{ x.name }}
      </li>
  </el-card>
</template>

<script>
import buptPoints from '../assets/buptPoints.json'

export default {
  name: "search-head",
  emits: ['get-route', 'setbeg', 'setdest', 'setpass'],
  props: {
    selected: Object,
    routing: Object
  },
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
      showRouteCard: '',
      currentInput: {},
      begin: {
        value: "start",
        legal: false,
        selected: false,
        type: "beg"
      },
      dest: {
        value: "",
        legal: false,
        selected: false,
        type: "dest"
      },
      passby: [],
      route: false,
    };
  },
  watch: {
    selected() {
      this.begin.value = "当前位置"
      this.begin.legal = true
    }
  },
  computed: {
    cardShow() {
      return !this.currentInput.selected && this.currentInput.value
    },
    stringQuery() {
      if (!this.currentInput.value) return null;
      let names = Object.keys(buptPoints);
      const getlist = (str) => {return names.filter((x) => {return x.includes(str)}).map(x=> { return {value:x,name:x} })}
      let result;
      var ds = /(\d*)班-(.*)课(-.*)?/
      let match = this.currentInput.value.match(ds)
      if (match) {
        let number = parseInt(match[1])
        let list = getlist('教学楼')
        result = [list[number%list.length]]
      } else {
        result = getlist(this.currentInput.value)
        if (this.currentInput.value === '食堂') {
          const color_map= [
            "#00b51b",
            "#97b500",
            "#c9bb34",
            "#d18726",
            "#d14826",
          ]
          result = result.map(x => {
            let load = Math.random()
            let color = Math.ceil(load*5)-1
            color = color_map[color]
            x.name = x.value + ` 负载: ${load.toString().slice(0,4)}`
            return {...x, load, style: {color: color}}
          })
        }       
        result.sort((x,y) => x.load-y.load)
      }
      return result
      // return [this.currentInput.value, "aaa"];
    },
    checkAllLegal() {
      let res = this.begin.legal && this.dest.legal
      for (let x of this.passby) {
        res &&= x.legal
      }
      return res
    },
  },
  methods: {
    setpoints(str) {
      switch (str) {
        case "beg":
          this.$emit('setbeg', this.begin.value)
          break;
        case "dest":
          this.$emit('setdest', this.dest.value)
          break;
        case "pass":
          this.$emit('setpass', this.passby.filter(x=>x.legal).map(x => x.value))
          break;
      }
    },
    checkInput() {
      this.currentInput.selected = false
      if(this.currentInput.value && this.stringQuery.length && (this.currentInput.value === this.stringQuery[0].value)) {
        this.currentInput.legal = true
        this.setpoints(this.currentInput.type)
      }
      else {
        this.currentInput.legal = false
      }
    },
    invRoute() {
      this.route = !this.route;
      this.currentInput = {};
    },
    changeInput(str) {
      this.currentInput.value = str;
      this.checkInput()
      this.currentInput.selected = true;
    },
    getRoute() {
      this.$emit('get-route')
    }
  },
  components: {
  }
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
.passby-scroll {
  overflow-y: auto;
  max-height: 30vh;
}
</style>
