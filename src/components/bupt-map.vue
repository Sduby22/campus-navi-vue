<template>
  <div id="map-div">
    <canvas id="map-canvas" />
  </div>
  <div class="center">
    {{ touch }}
  </div>
</template>

<script>
import "@interactjs/auto-start";
import "@interactjs/actions/drag";
import "@interactjs/actions/resize";
import "@interactjs/actions/gesture";
import "@interactjs/modifiers";
import "@interactjs/dev-tools";
import interact from "@interactjs/interact";
import { onMounted } from "@vue/runtime-core";
import bupt1 from "../assets/bupt.png";
import markerimg from "../assets/marker.png";
import directionimg from "../assets/direction.png";

export default {
  name: "bupt-map",
  setup() {
    var canvas = null;
    var ctx = null;

    class canvasImage {
      constructor(src, x = 0, y = 0, scale = 1) {
        this.img = new Image();
        this.load = new Promise((resolve, reject) => {
          this.img.onload = () => {
            resolve('ok')
          }
          this.img.onerror = () => {
            reject('error')
          }
        })
        this._x = x;
        this._y = y;
        this.scale = scale;
        this.img.src = src;
      }
      get x() {
        return this._x
      }
      get y() {
        return this._y
      }
      async draw() {
        let scale = this.scale;
        await this.load
        ctx.drawImage(
          this.img,
          this._x,
          this._y,
          this.img.width * scale,
          this.img.height * scale
        );
        return this.load
      }
    } 

    class DirectionImg extends canvasImage {
      constructor(src, x = 0, y = 0, scale = 1, angle = 0) {
        super(src, x, y, scale) 
        this.x = x
        this.y = y
        this.angle = angle
      }
      get x() {
        return this._x + this.img.width/2 * this.scale
      }
      get y() {
        return this._y + this.img.height/2 * this.scale
      }
      set x(x) {
        this._x = x - this.img.width/2 * this.scale
      }
      set y(y) {
        this._y = y - this.img.height/2 * this.scale 
      }
      async draw() {
        let scale = this.scale;
        await this.load
        ctx.save()
        ctx.translate(this.x, this.y)
        ctx.rotate(this.angle)
        ctx.drawImage(
          this.img,
          -0.5*this.img.width * scale,
          -0.5*this.img.height * scale,
          this.img.width * scale,
          this.img.height * scale
        );
        ctx.restore()
        return this.load
      }
    }

    class MarkerImg extends canvasImage {
      constructor(src, x = 0, y = 0, scale = 1) {
        super(src, x, y, scale) 
        this.x = x
        this.y = y
      }
      get x() {
        return this._x + this.img.width/2*this.scale
      }
      get y() {
        return this._y + this.img.height * this.scale
      }
      set x(x) {
        this._x = x - this.img.width/2*this.scale
      }
      set y(y) {
        this._y = y - this.img.height * this.scale 
      }
    }

    var bg = new canvasImage(bupt1);
    var marker = new MarkerImg(markerimg, 500, 500, 0.04)
    var direction = new DirectionImg(directionimg, 500, 600, 0.04, 0)

    const loadcanvas = () => {
      canvas = document.getElementById("map-canvas");
      ctx = canvas.getContext("2d");
      canvasResize();
      window.onresize = canvasResize;
    };

    const canvasResize = () => {
      let dpr = window.devicePixelRatio;
      canvas.height = window.innerHeight * dpr;
      canvas.width = window.innerWidth * dpr;
      canvas.style.height = window.innerHeight;
      canvas.style.width = window.innerWidth;
      ctx.imageSmoothingEnabled = true;
      draw()
    };

    const draw = async () => {
      await bg.draw();
      await marker.draw()
      await direction.draw()
    };

    onMounted(loadcanvas);

    return {
      canvasResize,
      canvas,
      ctx,
    };
  },
  mounted() {
    interact;
  },
  methods: {},
  data() {
    return {
      touch: "",
    };
  },
};
</script>

<style lang="scss" scoped>
.map_img {
  /* max-width: 100vw; */
}
.center {
  position: absolute;
  top: 50vh;
  left: 50vw;
}
</style>
