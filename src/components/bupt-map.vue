<template>
  <div class="map_img">
    <canvas
      class="map_img"
      id="map-canvas"
      @mousedown="mouseDown"
      @mousemove="mouseMove"
      @mouseup="mouseUp"
      @mouseleave="mouseUp"
      @mousewheel="mouseWheel"
      @touchstart="touchStart"
      @touchmove="touchMove"
      @touchend="touchEnd"
    >
    </canvas>
  </div>
  <!-- <div class="center">{{ dpr }}</div> -->
</template>

<script>
import buptimg from "../assets/bupt.png";

export default {
  name: "bupt-map",
  mounted() {
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
    img2.src = buptimg;
    return {
      img1,
      img2,
    };
  },
  data() {
    return {
      dpr: window.devicePixelRatio,
      imgs: [
        {
          lastcoords: [0, 0],
          newcoords: [0, 0],
          img: this.img1,
        },
      ],
      store: {
        X1: null,
        Y1: null,
        X2: null,
        Y2: null,
        startX: null,
        startY: null,
      },
      canvas: null,
      ctx: null,
    };
  },
  methods: {
    // mouseWheel(event) {

    // },
    mouseDown(event) {
      this.ctx.save();
      this.startX = event.offsetX;
      this.startY = event.offsetY;
      this.X1 = event.offsetX;
      this.Y1 = event.offsetY;
      for (let img of this.imgs) {
        img.newcoords[0] = img.lastcoords[0] - this.X1;
        img.newcoords[1] = img.lastcoords[1] - this.Y1;
      }
      this.ctx.translate(this.X1, this.Y1);
    },
    touchStart(event) {
      event.preventDefault();
      this.ctx.save();
      this.startX = event.touches[0].pageX;
      this.startY = event.touches[0].pageY;
      this.X1 = event.touches[0].pageX;
      this.Y1 = event.touches[0].pageY;
      for (let img of this.imgs) {
        img.newcoords[0] = img.lastcoords[0] - this.X1;
        img.newcoords[1] = img.lastcoords[1] - this.Y1;
      }
      this.ctx.translate(this.X1, this.Y1);
    },
    mouseMove(event) {
      if (this.startX) {
        let deltaX = event.offsetX - this.X1;
        let deltaY = event.offsetY - this.Y1;
        this.X1 = event.offsetX;
        this.Y1 = event.offsetY;
        this.ctx.translate(deltaX, deltaY);
        this.draw();
      }
    },
    touchMove(event) {
      if (event.touches.length === 1) {
        if (this.startX) {
          let deltaX = event.touches[0].pageX - this.X1;
          let deltaY = event.touches[0].pageY - this.Y1;
          this.X1 = event.touches[0].pageX;
          this.Y1 = event.touches[0].pageY;
          this.ctx.translate(deltaX, deltaY);
          this.draw();
        }
      } else {
        event;
      }
    },
    mouseUp(event) {
      if (this.startX) {
        for (let img of this.imgs) {
          let arr = [event.offsetX - this.startX, event.offsetY - this.startY];
          img.lastcoords[0] += arr[0];
          img.lastcoords[1] += arr[1];
        }
        this.startX = null;
        this.ctx.restore();
      }
    },
    touchEnd() {
      if (this.startX) {
        for (let img of this.imgs) {
          let arr = [this.X1 - this.startX, this.Y1 - this.startY];
          img.lastcoords[0] += arr[0];
          img.lastcoords[1] += arr[1];
        }
        this.startX = null;
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
      let dpr = window.devicePixelRatio
      this.canvas.width *= dpr
      this.canvas.height *= dpr
      this.ctx.scale(dpr,dpr);
      // this.clearCanvas()
      // this.draw()
    },
    draw() {
      this.clearCanvas();
      for (let img of this.imgs) {
        this.ctx.drawImage(img.img, img.newcoords[0], img.newcoords[1]);
      }
    },
    update() {
      this.clearCanvas();
      this.draw();
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
</style>
