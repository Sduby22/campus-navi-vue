<template>
  <el-button type="primary" size="medium" @click="click"></el-button>
</template>

<script>
import axios from 'axios'

export default {
  setup() {
    const click = async () => {
      axios.get('http://localhost:8081/asd').then((data) => {
        console.log(data)
      })
    }
    return {
      click
    }
  } 
}
</script>

<style scoped>

</style>
