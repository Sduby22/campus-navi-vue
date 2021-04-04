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
    <el-slider
      v-model="scale"
      @change="sliderupdate"
      :min="0.3"
      :max="1"
      :step="0.05"
      vertical
      height="200px"
    >
    </el-slider>
  </div>
  <!-- <div class="center">{{ dpr }}</div> -->
</template>

<script>
import buptimg from "../assets/bupt.png";
import logo from "../assets/logo.png";

export default {
  props: {
    routing: Object
  },
  name: "bupt-map",
  mounted() {
    this.mainImg = this.imgs[0]
    this.canvas = document.getElementById("map-canvas");
    this.ctx = this.canvas.getContext("2d");
    this.img1.onload = () => {
      this.draw();
    };
    this.canvasResize();
    window.onresize = () => {
      this.canvasResize();
      this.draw();
    };
  },
  setup() {
    let img1 = new Image();
    let img2 = new Image();
    img1.src = buptimg;
    img2.src = logo;
    return {
      img1,
      img2,
    };
  },
  data() {
    return {
      scale: 1,
      dpr: window.devicePixelRatio,
      imgs: [
        {
          lastcoords: [0, 0],
          newcoords: [0, 0],
          img: this.img1,
        },
        {
          lastcoords: [100, 100],
          newcoords: [100, 100],
          img: this.img2,
        },
      ],
      mainImg: null,
      store: {
        last: null,
        start: null,
      },
      canvas: null,
      ctx: null,
    };
  },
  methods: {
    mouseWheel(event) {
      let img = this.mainImg;
      let offset = [event.offsetX, event.offsetY]
      img.newcoords[0] = (-offset[0]+img.lastcoords[0])/this.scale;
      img.newcoords[1] = (-offset[1]+img.lastcoords[1])/this.scale;
      this.scale += event.deltaY < 0 ? 0.05 : -0.05;
      if (this.scale > 1) this.scale = 1;
      else if (this.scale < 0.5) this.scale = 0.5;
      img.newcoords[0] *= this.scale
      img.newcoords[1] *= this.scale
      this.ctx.save()
      this.ctx.translate(offset[0], offset[1])
      this.draw();
      this.ctx.restore()
      img.lastcoords = [img.newcoords[0] + offset[0], img.newcoords[1] + offset[1]]
    },
    logStart(e) {
      this.ctx.save();
      let start = [];
      if (e.touches) start = [e.touches[0].pageX, e.touches[0].pageY];
      else start = [e.offsetX, e.offsetY];
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
    logEnd() {
      if (this.start) {
        // let offset = [];
        // if (e.touches) offset = [e.touches[0].pageX, e.touches[0].pageY];
        // else offset = [e.offsetX, e.offsetY];
        let img = this.mainImg;
        let arr = this.toRelative(this.last, this.start);
        img.lastcoords[0] += arr[0];
        img.lastcoords[1] += arr[1];
        this.start = null;
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
      this.ctx.drawImage(
        img.img,
        img.newcoords[0],
        img.newcoords[1],
        img.img.width * scale,
        img.img.height * scale
      );
      let nw = this.mainImg.newcoords
      this.ctx.fillRect(nw[0], nw[1], 10, 10)
      if (this.start) {
        let c = this.toCanvas([ this.start[0] - this.mainImg.lastcoords[0], this.start[1] - this.mainImg.lastcoords[1] ])
        this.ctx.fillRect(c[0], c[1], 10, 10)
      }
    },
    sliderupdate() {
      let img = this.mainImg;
      img.newcoords[0] = img.lastcoords[0];
      img.newcoords[1] = img.lastcoords[1];
      this.clearCanvas();
      this.draw();
    },
    toCanvas([x,y]) {
      let nw = this.mainImg.newcoords
      let sc = this.scale
      return [nw[0] + x*sc,nw[1] + y*sc]
    },
    toRelative([x1, y1], [x2, y2]) {
      return [x1 - x2, y1 - y2];
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
  position: fixed;
  bottom: 40px;
  right: 20px;
}
</style>
