<template>
  <div>
    <img
      class="map_img"
      src="../assets/bupt.png"
      alt="bupt"
      ondragstart="return false;"
      oncontextmenu="return false;"
      @touchmove="showtouch"
    />
  </div>
  <div class="center">
    {{ touch }} 
  </div>
</template>

<script>
export default {
  name: 'bupt-map',
  methods: {
    showtouch(event) {
      this.touch=event.touches[0].pageX
    }
  },
  data() {
    return {
      touch:'',
    }
  }
}
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
