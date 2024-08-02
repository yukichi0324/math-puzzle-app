<!-- src/App.vue -->
<template>
  <div id="app">
    <PuzzleGrid />
  </div>
</template>

<script>
import PuzzleGrid from './components/PuzzleGrid.vue'

export default {
  name: 'App',
  components: {
    PuzzleGrid
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
}
</style>
