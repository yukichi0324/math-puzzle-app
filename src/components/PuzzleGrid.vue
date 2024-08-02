<!-- src/components/PuzzleGrid.vue -->
<template>
    <div class="grid-container">
      <div class="grid">
        <div v-for="n in 48" :key="n" class="grid-cell"></div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    name: 'PuzzleGrid'
  }
  </script>
  
  <style scoped>
  .grid-container {
    display: flex;
    justify-content: center;
    align-items: center;
  }
  
  .grid {
    display: grid;
    grid-template-columns: repeat(6, 50px);
    grid-template-rows: repeat(8, 50px);
    gap: 5px;
  }
  
  .grid-cell {
    width: 50px;
    height: 50px;
    background-color: lightgray;
    border: 1px solid lightgray;
  }
  </style>
  