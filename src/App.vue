<template>
  <div id="app">
    <Game :cubeValues="cubeValues" :wordLength="wordLength"/>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import Game from './components/Game.vue'

export default Vue.extend({
  name: 'App',
  components: { Game },
  data: () => ({
    cubeValues: genCubeValues(10),
    wordLength: 6
  })
})

function genCubeValues (setNum: number): string[][] {
  const alpha = [
    'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M',
    'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'
  ]
  const sets: string[][] = []
  for (let s = 0; s < setNum; s++) {
    const set: string[] = []
    const _alpha = [...alpha]
    for (let i = 0; i < 6; i++) {
      const letter = _alpha[Math.floor(Math.random() * _alpha.length)]
      _alpha.splice(_alpha.indexOf(letter), 1)
      set.push(letter)
    }
    sets.push(set)
  }
  return sets
}
</script>

<style>

body {
  font-family: Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: #2c3e50;
}

</style>
