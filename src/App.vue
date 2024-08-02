<!-- src/App.vue -->
<template>
  <div id="app" :style="appStyle">
    <PuzzleGrid @update-score="handleScoreUpdate"/>
  </div>
</template>

<script>
import PuzzleGrid from './components/PuzzleGrid.vue'

export default {
  name: 'App',
  components: {
    PuzzleGrid
  },
  data() {
    return {
      appScore: 0,
      backgroundImage: require('@/assets/nohara.png') // 初期の背景画像
    }
  },
  computed: {
    appStyle() {
      return {
        backgroundImage: `url(${this.backgroundImage})`,
        backgroundSize: 'cover',
        backgroundPosition: 'center',
        height: '100vh',
        width: '100vw',
        paddingTop: '60px'
      };
    }
  },
  methods: {
    handleScoreUpdate(newScore) {
      console.log('***************************');
      console.log(newScore);
      console.log('***************************');

      this.appScore = newScore;
      
      if (newScore >= 2) {
        this.backgroundImage = require('@/assets/yama-no-ue.png');
      } else {
        this.backgroundImage = require('@/assets/nohara.png'); // スコアが条件を満たさない場合、元の画像に戻す
      }
    }
  }
}
</script>

<style>
html, body {
  margin: 0;
  padding: 0;
  height: 100%;
  overflow: hidden;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
