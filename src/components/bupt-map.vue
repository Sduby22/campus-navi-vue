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
        this.x = x;
        this.y = y;
        this.scale = scale;
        this.img.src = src;
      }
      async draw() {
        let scale = this.scale;
        await this.load
        ctx.drawImage(
          this.img,
          this.x,
          this.y,
          this.img.width * scale,
          this.img.height * scale
        );
      }
    }

    var bgImg = new canvasImage(bupt1);

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
      draw()
    };

    const draw = () => {
      bgImg.draw();
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
