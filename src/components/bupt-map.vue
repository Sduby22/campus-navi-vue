<template>
  <div class="map_img">
    <canvas
      class="map_img"
      id="map-canvas"
      @mousedown="logStart"
      @mousemove="logMove"
      @mouseup="logEnd"
      @mouseleave="logEnd"
      @mousewheel="mouseWheel"
      @touchstart="logStart"
      @touchmove="logMove"
      @touchend="logEnd"
    >
    </canvas>
  </div>
  <div class="block">
    <el-card shadow="always" :body-style="{padding: '10px 0'}">
      <div style="position: relative" class="slider">
        <el-slider
          v-model="slidescale"
          @change="sliderupdate"
          :min="minScale"
          :max="maxScale"
          :step="0.05"
          vertical
          height="200px"
        > </el-slider>
        <i style="position:absolute;top:10px;left:-10px" class="el-icon-plus"></i>
        <i style="position:absolute;bottom:10px;left:-10px" class="el-icon-minus"></i>
      </div>
    </el-card>
      <div class="button">
        <el-button type="primary" size="medium" @click="switchButton"
          >切换到{{ button }}</el-button
        >
      </div>
  </div>
  <!-- <div class="center">{{ dpr }}</div> -->
</template>

<script>
import buptimg from "../assets/bupt.png";
import shahe from "../assets/bupt2.png";

export default {
  props: {
    routing: Object,
  },
  name: "bupt-map",
  mounted() {
    this.mainImg = this.imgs[0];
    this.canvas = document.getElementById("map-canvas");
    this.ctx = this.canvas.getContext("2d");
    this.img1.onload = () => {
      this.draw();
    };
    this.img2.onload = () => {
      console.log("2");
    };
    this.canvasResize();
    window.onresize = () => {
      this.canvasResize();
      this.draw();
    };
  },
  setup() {
    const minScale = 0.3;
    const maxScale = 1.4;
    let img1 = new Image();
    let img2 = new Image();
    img1.src = buptimg;
    img2.src = shahe;
    return {
      img1,
      img2,
      maxScale,
      minScale,
    };
  },
  data() {
    return {
      slidescale: 1,
      scale: 1,
      dpr: window.devicePixelRatio,
      mainImg: { name: "" },
      button: "沙河校区",
      imgs: [
        {
          name: "本部",
          lastcoords: [0, 0],
          newcoords: [0, 0],
          img: this.img1,
        },
        {
          name: "沙河校区",
          lastcoords: [100, 100],
          newcoords: [100, 100],
          img: this.img2,
        },
      ],
      lastclick: [0, 0],
    };
  },
  methods: {
    switchButton() {
      this.switchMap(this.button);
      this.button = this.button === "本部" ? "沙河校区" : "本部";
    },
    switchMap(name) {
      for (let img of this.imgs) {
        if (img.name === name) {
          this.mainImg = img;
          img.newcoords = img.lastcoords;
          this.draw();
          break;
        }
      }
    },
    zoom(center, scale) {
      let img = this.mainImg;
      img.newcoords[0] = (-center[0] + img.lastcoords[0]) / this.scale;
      img.newcoords[1] = (-center[1] + img.lastcoords[1]) / this.scale;
      this.scale = scale;
      img.newcoords[0] *= scale;
      img.newcoords[1] *= scale;
      this.ctx.save();
      this.ctx.setTransform(1, 0, 0, 1, 0, 0);
      this.ctx.translate(center[0], center[1]);
      this.draw();
      this.ctx.restore();
      img.lastcoords = [
        img.newcoords[0] + center[0],
        img.newcoords[1] + center[1],
      ];
    },
    mouseWheel(event) {
      let offset = [event.offsetX, event.offsetY];
      let scale = this.scale + (event.deltaY < 0 ? 0.05 : -0.05);
      if (scale > this.maxScale) scale = this.maxScale;
      else if (scale < this.minScale) scale = this.minScale;
      this.zoom(offset, scale);
    },
    logStart(e) {
      this.ctx.save();
      let start = [];
      if (e.touches) {
        e.preventDefault();
        start = [e.touches[0].pageX, e.touches[0].pageY];
      } else start = [e.offsetX, e.offsetY];
      this.start = start;
      this.last = start;
      let img = this.mainImg;
      img.newcoords = this.toRelative(img.lastcoords, this.last);
      this.ctx.translate(this.last[0], this.last[1]);
    },
    logMove(e) {
      if (this.start) {
        let offset = [];
        if (e.touches) offset = [e.touches[0].pageX, e.touches[0].pageY];
        else offset = [e.offsetX, e.offsetY];
        let delta = this.toRelative(offset, this.last);
        this.last = offset;
        this.ctx.translate(delta[0], delta[1]);
        this.draw();
      }
    },
    logEnd(e) {
      if (this.start) {
        let offset = this.last;
        this.lastclick = offset;
        let img = this.mainImg;
        let arr = this.toRelative(this.last, this.start);
        img.lastcoords[0] += arr[0];
        img.lastcoords[1] += arr[1];
        if (!e.touches || e.touches.length === 0) this.start = null;
        this.ctx.restore();
      }
    },
    clearCanvas() {
      let ctx = this.ctx;
      ctx.save();
      ctx.setTransform(1, 0, 0, 1, 0, 0);
      ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
      ctx.restore();
    },
    canvasResize() {
      this.canvas.height = window.innerHeight;
      this.canvas.width = window.innerWidth;
      let dpr = window.devicePixelRatio;
      this.canvas.width *= dpr;
      this.canvas.height *= dpr;
      this.ctx.scale(dpr, dpr);
    },
    draw() {
      this.clearCanvas();
      let img = this.mainImg;
      let scale = this.scale || 1;
      console.log('new ', img.newcoords);
      console.log('scale ', this.scale);
      this.ctx.drawImage(
        img.img,
        img.newcoords[0],
        img.newcoords[1],
        img.img.width * scale,
        img.img.height * scale
      );
      let nw = this.mainImg.newcoords;
      this.ctx.fillRect(nw[0], nw[1], 10, 10);
      if (this.start) {
        let c = this.toCanvas([
          (this.start[0] - this.mainImg.lastcoords[0]) / this.scale,
          (this.start[1] - this.mainImg.lastcoords[1]) / this.scale,
        ]);
        this.ctx.fillRect(c[0], c[1], 10, 10);
      }
    },
    sliderupdate() {
      this.zoom(this.lastclick, this.slidescale);
      this.slidescale = this.scale;
    },
    toCanvas([x, y]) {
      let nw = this.mainImg.newcoords;
      let sc = this.scale;
      return [nw[0] + x * sc, nw[1] + y * sc];
    },
    toRelative([x1, y1], [x2, y2]) {
      return [x1 - x2, y1 - y2];
    },
    getDistance(p1, p2) {
      return Math.sqrt(Math.pow(p1[0] - p2[0], 2) + Math.pow(p1[1] - p2[1], 2));
    },
  },
};
</script>

<style lang="scss" scoped>
.map_img {
  width: 100vw;
  height: 100vh;
}
.center {
  position: absolute;
  top: 50vh;
  left: 50vw;
}
.btm {
  position: fixed;
  bottom: 10vh;
  right: 10vh;
}
.block {
  width: 60px;
  display:flex;
  flex-direction: column;
  :deep(.el-card__body) {
    display: flex;
    align-items: flex-end;
    justify-content: space-between;
    flex-direction: column;
  };
  position: fixed;
  bottom: 40px;
  right: 20px;
}
.button {
  align-self: flex-end;
  margin-top: 20px;
}
</style>
