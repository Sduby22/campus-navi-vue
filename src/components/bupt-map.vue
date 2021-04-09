<template>
  <div id="map-div">
    <canvas id="map-canvas" @wheel="wheelzoom" />
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
  props: {
    routing: Object
  },
  setup() {
    const minScale = 0.3
    const maxScale = 1.7
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
        // for background, _x _y is the real coordinate.
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
      _bgmove (offsetX, offsetY) {
        this._x += offsetX
        this._y += offsetY
      }
      _bgzoom (centerX, centerY, ds) {
        let oldscale = this.scale
        this.scale += ds
        this.scale = this.scale > maxScale ? maxScale : this.scale
        this.scale = this.scale < minScale ? minScale : this.scale
        this._x = (this._x - centerX) * (this.scale / oldscale) + centerX
        this._y = (this._y - centerY) * (this.scale / oldscale) + centerY
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

    class relativeImage extends canvasImage {
      constructor(src, x=0, y=0, scale=1, angle=0) {
        super(src, x, y, scale)
        this.relativeX = x
        this.relativeY = y
        this.angle = angle
      }
      // _x and _y are relative to original size of bg.
      get relativeX() {
        return this._x
      }
      get relativeY() {
        return this._y
      }
      set relativeX(x) {
        this._x = x
      }
      set relativeY(y) {
        this._y = y
      }
      get x() {
        return bg.x + this._x * bg.scale
      }
      get y() {
        return bg.y + this._y * bg.scale
      }
      set x(x) {
        this._x = (x - bg.x) / bg.scale
      }
      set y(y) {
        this._y = (y - bg.y) / bg.scale
      }
      async draw(offsetX, offsetY) {
        let scale = this.scale;
        await this.load
        ctx.save()
        ctx.translate(this.x, this.y)
        ctx.rotate(this.angle)
        ctx.drawImage(
          this.img,
          offsetX,
          offsetY,
          this.img.width * scale,
          this.img.height * scale
        );
        ctx.restore()
        return this.load
      }
    }

    class DirectionImg extends relativeImage {
      constructor(src, x = 0, y = 0, scale = 1, angle = 0) {
        super(src, x, y, scale, angle) 
      }
      async draw() {
        return super.draw(-0.5*this.img.width*this.scale,-0.5*this.img.height*this.scale)
      }
    }

    class MarkerImg extends relativeImage {
      constructor(src, x = 0, y = 0, scale = 1, angle = 0) {
        super(src, x, y, scale, angle) 
      }
      async draw() {
        return super.draw(-0.5*this.img.width*this.scale, -this.img.height*this.scale)
      }
    }
    var buptimg1 = new canvasImage(bupt1, 0, 0, 1);
    var bg = buptimg1
    var marker = new MarkerImg(markerimg, 500, 500, 0.04)
    var direction = new DirectionImg(directionimg, 500, 600, 0.04, 0)
    var direction2 = new DirectionImg(directionimg, 500, 600, 0.04, 45)
    direction2.relativeX = 500
    direction2.relativeY = 600

    var renderList = [bg, marker, direction, direction2]

    const loadcanvas = () => {
      canvas = document.getElementById("map-canvas");
      ctx = canvas.getContext("2d");
      canvasResize();
      window.onresize = canvasResize;
    };

    const canvasResize = () => {
      let dpr = window.devicePixelRatio;
      console.log(dpr)
      canvas.height = window.innerHeight * dpr;
      canvas.width = window.innerWidth * dpr;
      ctx.scale(dpr, dpr)
      ctx.imageSmoothingEnabled = true;
      draw()
    };

    const clearCanvas = () => {
      ctx.clearRect(0, 0, canvas.width, canvas.height)
    }
    const draw = async () => {
      clearCanvas()
      for (let item of renderList) {
        await item.draw()
      }
    };

    const loadInteractjs = () => {
      interact('#map-canvas').draggable({
        listeners: {
          move(e) {
            bg._bgmove(e.dx, e.dy) 
            draw()
          }
        },
      }).gesturable({
        listeners: {
          move(e) {
            bg._bgmove(e.dx, e.dy)
            bg._bgzoom(e.x0, e.y0, e.ds)
            draw()
          }
        }
      })
    }

    onMounted(() => {
      loadcanvas()
      loadInteractjs()
    });

    return {
      canvasResize,
      canvas,
      ctx,
    };
  },

  mounted() {
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
  #map-canvas {
    height: 100%;
    width: 100%;
    touch-action: none;
  }

  .center {
    position: absolute;
    top: 50vh;
    left: 50vw;
  }
</style>
