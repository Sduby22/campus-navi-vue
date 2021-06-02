<template>
  <div id="map-div">
    <canvas
      id="map-canvas"
      @mousedown="setdown"
      @mousemove="setmove"
      @mouseup="choosePoint"
      @wheel="wheelZoom"
    />
  </div>
  <div class="bottom-tool">
    <div style="width: 30px">
      <el-slider v-model="speed" vertical :max="500" :min="10" height="200px">
      </el-slider>
    </div>
    <el-button type="primary" size="medium" @click="switchButton"
      >切换到{{ currentMap }}</el-button
    >
  </div>
</template>

<script>
import { ref, watch } from 'vue'
import "@interactjs/auto-start";
import "@interactjs/actions/drag";
import "@interactjs/actions/resize";
import "@interactjs/actions/gesture";
import "@interactjs/modifiers";
import "@interactjs/dev-tools";
import interact from "@interactjs/interact";
import { onMounted } from "@vue/runtime-core";
import bupt1 from "../assets/bupt.png";
import bupt2 from "../assets/bupt2.png";
import markerimg from "../assets/marker.png";
import directionimg from "../assets/direction.png";
  

export default {
  name: "bupt-map",
  props: {
    routing: Object,
    start: Boolean,
  },
  emits: ['choose-point'],
  setup(props, {emit}) {
    const color_map = [
      "#d62700",
      "#ff5e00",
      "#ffb433",
      "#e8dc00",
      "#0ed900"
    ]
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
      constructor(x = 0, y = 0, scale = 0.04, angle = 0) {
        super(directionimg, x, y, scale, angle) 
      }
      async draw() {
        return super.draw(-0.5*this.img.width*this.scale,-0.5*this.img.height*this.scale)
      }
    }

    class MarkerImg extends relativeImage {
      constructor(x = 0, y = 0, scale = 0.04, angle = 0) {
        super(markerimg, x, y, scale, angle) 
      }
      async draw() {
        return super.draw(-0.5*this.img.width*this.scale, -this.img.height*this.scale)
      }
    }

    class LineImg {
      constructor(x1, y1, x2, y2, begin=0, end=0, thickness=3, color='#00aa00') {
        const xoffset = 0
        const yoffset = 0
        this._x1 = x1 + (x2-x1)*begin + xoffset
        this._x2 = x2 - (x2-x1)*end+ xoffset
        this._y1 = y1 + (y2-y1)*begin + yoffset
        this._y2 = y2 - (y2-y1)*end+ yoffset
        this.color = color
        this.thickness = thickness
      }
      get x1() {
        return bg.x + this._x1 * bg.scale
      }
      get x2() {
        return bg.x + this._x2 * bg.scale
      }
      get y1() {
        return bg.y + this._y1 * bg.scale
      }
      get y2() {
        return bg.y + this._y2 * bg.scale
      }
      set x1(v) {
      }
      set x2(v) {
      }
      set y1(v) {
      }
      set y2(v) {
      }
      async draw() {
        ctx.beginPath()
        ctx.moveTo(this.x1, this.y1)
        ctx.lineTo(this.x2, this.y2)
        ctx.lineWidth = this.thickness
        ctx.strokeStyle = this.color
        ctx.stroke()
        return Promise.resolve()
      }
    }

    var buptimg1 = new canvasImage(bupt1, 0, 0, 1);
    var buptimg2 = new canvasImage(bupt2, 0, 0, 1);
    var bg = buptimg1

    var map1 = {
      map: buptimg1,
      markers: [],
      lines: [],
      directions: [],
    }

    var map2 = {
      map: buptimg2,
      markers: [],
      lines: [],
      directions: [],
    }

    let lines = ref([])
    lines.value=map1.lines
    let markers = ref([])
    let directions = ref([])
    markers.value=map1.markers
    directions.value = map1.directions

    var renderList = [lines, markers, directions]

    const toRelativeY = (y) => {
      return (y-bg.y)/bg.scale
    }

    const toRelativeX = (x) => {
      return (x-bg.x)/bg.scale
    }

    var dragging = false
    const setdown = () => {
      dragging = false
    }

    const choosePoint = (e) => {
      if (dragging)
        return
      let p = [toRelativeX(e.offsetX), toRelativeY(e.offsetY)]
      emit('choose-point', {coor:p, isshahe: bg===buptimg2})
    }

    const setmove = () => {
      dragging = true
    }

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
      ctx.scale(dpr, dpr)
      ctx.imageSmoothingEnabled = true;
      draw()
    };

    const clearCanvas = () => {
      ctx.clearRect(0, 0, canvas.width, canvas.height)
    }

    const draw = async () => {
      clearCanvas()
      await bg.draw()
      for (let group of renderList) {
        for (let item of group.value) {
          await item.draw()
        }
      }
    };

    const loadInteractjs = () => {
      var zoomx, zoomy
      interact('#map-canvas').draggable({
        listeners: {
          move(e) {
            bg._bgmove(e.dx, e.dy) 
            draw()
          }
        },
      }).gesturable({
        listeners: {
          start(e) {
            zoomx = e.x0
            zoomy = e.y0
          },
          move(e) {
            zoomx += e.dx
            zoomy += e.dy
            bg._bgmove(e.dx, e.dy)
            bg._bgzoom(zoomx, zoomy, e.ds)
            draw()
          }
        }
      })
    }
    
    const wheelZoom = (e) => {
      let ds = 0
      if(e.deltaY > 0)
        ds = -0.05
      else if(e.deltaY < 0)
        ds = 0.05
      bg._bgzoom(e.offsetX, e.offsetY, ds)
      draw()
    }

    var currentMap = ref('沙河')
    const switchButton = () => {
      /* renderList = [] */
      currentMap.value = '本部'
      if (bg === buptimg1) {
        return switchMap(map2)
      } else {
        currentMap.value = '沙河'
        return switchMap(map1)
      }
    }

    const switchMap = (mapobj) => {
      bg = mapobj.map
      markers.value = mapobj.markers
      lines.value = mapobj.lines
      directions.value = mapobj.directions
      draw()
    }

    onMounted(() => {
      loadcanvas()
      loadInteractjs()
    });

    const setMarkers = (obj) => {
      if (obj) {
        if (obj.isshahe)
          map2.markers.push(new MarkerImg(obj.coor[0],obj.coor[1]))
        else
          map1.markers.push(new MarkerImg(obj.coor[0],obj.coor[1]))
      }
    }

    const setLines = (data) => {
      let line;
      for (let x in data.path) {
        let x1 = data.path[x].start[0]
        let y1 = data.path[x].start[1]
        let x2 = data.path[x].end[0]
        let y2 = data.path[x].end[1]
        if (x == 0) {
          line = new LineImg( x1, y1, x2, y2, data.path[x].arg )
        }
        else if (x == data.path.length-1) {
          line = new LineImg( x1, y1, x2, y2, 0, 1-data.path[x].arg )
        }
        else {
          line = new LineImg( x1, y1, x2, y2 )
        }
        line.capacity = data.path[x].capacity
        line.color = color_map[Math.ceil(data.path[x].capacity*5)-1]
        if (data.shahe) {
          map2.lines.push(line)
        } else {
          map1.lines.push(line)
        }
      }
    }

    var speed = ref(10)
    let time;
    let currPath, currPathObj
    let currmap
    let curMax
    const stopNavi = () => {
      time = null
      map1.directions.length =  0
      map2.directions.length =  0
    }

    var i = 0
    const step = (currentTime) => {
      console.log("step");
      let speedv = speed.value
      if (!time) {
        currPathObj = 0
        time = currentTime
        currmap = props.routing.path[currPathObj].shahe ? map2 : map1
        currPath = currmap.lines[0]
        map1.currIndex = 0
        map2.currIndex = 0
        curMax = props.routing.path[currPathObj].path.length
        i=0
        currmap.directions.push(new DirectionImg(currPath._x1, currPath._x2))
      }
      const deltatime = currentTime - time
      time = currentTime
      let deltaDistance = deltatime * speedv * 0.001 * currPath.capacity
      while (deltaDistance >= 0) {
        const tan = (currPath._y2 -currPath._y1)/(currPath._x2 -currPath._x1)
        let angle = Math.atan(tan)
        if (currPath._x2-currPath._x1<0)
          angle += Math.PI
        let deltax = deltaDistance * Math.cos(angle)
        let deltay = deltaDistance * Math.sin(angle)
        let exceedx = deltax - (currPath._x2-currPath._x1)
        if (exceedx * deltax <= 0) {
          // still in the same path
          deltaDistance = -1 
          currPath._x1 += deltax
          currPath._y1 += deltay
          currmap.directions[0].angle = Math.PI/2 + angle
          currmap.directions[0]._x = currPath._x1
          currmap.directions[0]._y = currPath._y1
        } else {
          // change path or might change campus
          deltaDistance = exceedx / Math.cos(angle)
          currPath.thickness = 1e-6
          currmap.currIndex += 1
          i += 1
          if (i == curMax) {
            // next path obj (may change campus)
            currPathObj += 1
            if (currPathObj == props.routing.path.length) {
              console.log("finished");
              stopNavi()
              return
              // finished navi
            } else {
              currmap.directions.length = 0
              currmap = props.routing.path[currPathObj].shahe ? map2 : map1
              currPath = currmap.lines[currmap.currIndex]
              currmap.directions.push(new DirectionImg(currPath._x1, currPath._x2))
              curMax = props.routing.path[currPathObj].path.length
              i = 0
            }
          } else {
            currPath = currmap.lines[currmap.currIndex]
          }
        } 
      }
      draw()
      if (props.start)
        requestAnimationFrame(step)
    }

    const startNavi = () => {
      requestAnimationFrame(step)
    }

    watch(props, () => {
      if (props.start) startNavi()
      else stopNavi()
    })

    watch(props.routing, () => {
      map1.markers.length = 0
      map2.markers.length = 0
      map1.lines.length = 0
      map2.lines.length = 0
      props.routing.pass.forEach(x => setMarkers(x))  
      setMarkers(props.routing.beg)
      setMarkers(props.routing.dest)
      props.routing.path.forEach(x => setLines(x))
      draw()
      console.log(props.routing);
    })

    return {
      speed,
      LineImg,
      choosePoint,
      wheelZoom,
      currentMap,
      switchButton,
      buptimg1,
      buptimg2,
      setdown,
      setmove,
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

.bottom-tool {
  position: fixed;
  bottom: 20px;
  right: 20px;
}
</style>
