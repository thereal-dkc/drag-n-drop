<template>
  <div id="app">
    <DragNDrop/>
  </div>
</template>

<script>
import DragNDrop from '@/components/DragNDrop.vue'

export default {
  name: 'App',
  components: {
    DragNDrop
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  height: 800px;
}
</style>
