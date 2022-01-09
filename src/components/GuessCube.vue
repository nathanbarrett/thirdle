<template>
  <div draggable="true" :id="'guessCube' + number" ref="guessCube" class="guess-cube">
    <div class="letters-hint top" :style="{opacity: !dragging && mouseOver ? 1 : 0}">
      <span v-for="index in 3" :key="index" class="letter-hint">{{ letters[index -1] }}</span>
    </div>
    <div class="scene" @click="nextLetter">
      <div class="cube" :class="'show-' + selectedFace">
        <div v-for="(face, index) in faces"
             :key="face"
             class="cube__face"
             :class="'cube__face--' + face"
             v-show="showCubeFace(letters[index])">
          {{ letters[index] }}
        </div>
      </div>
    </div>
    <div class="letters-hint bottom" :style="{opacity: !dragging && mouseOver ? 1 : 0}">
      <span v-for="index in 3" :key="index" class="letter-hint">{{ letters[index + 2] }}</span>
    </div>
    <div class="cube-number">{{ number }}</div>
  </div>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue'
export default Vue.extend({
  props: {
    letters: {
      type: Array as PropType<string[]>,
      required: true
    },
    number: {
      type: Number,
      required: true
    }
  },
  data: () => ({
    letter: '',
    dragging: false,
    mouseOver: false,
    faces: ['front', 'back', 'right', 'left', 'top', 'bottom']
  }),
  computed: {
    selectedFace () : string {
      return this.faces[this.letters.indexOf(this.letter)]
    }
  },
  watch: {
    value (val: string) {
      if (val && this.letter && val !== this.letter) {
        this.updateSelectedLetter(val)
      }
    }
  },
  methods: {
    nextLetter () {
      const index = this.letters.indexOf(this.letter)
      this.updateSelectedLetter(this.letters[index >= 5 ? 0 : index + 1])
    },
    updateSelectedLetter (letter: string) {
      this.letter = letter
      this.$emit('letterUpdated', this.number, this.letter)
    },
    onMouseDown () {
      console.log('dragging ')
      this.dragging = true
    },
    onMouseUp () {
      this.dragging = false
    },
    onMouseOver () {
      this.mouseOver = true
    },
    onMouseLeave () {
      this.mouseOver = false
      this.dragging = false
    },
    onDrop () {
      console.log('dropped')
      this.removeEventListeners()
      this.addEventListeners()
    },
    preventDefault (e: DragEvent) {
      e.preventDefault()
    },
    showCubeFace (letter: string): boolean {
      return !this.dragging || letter === this.letter
    },
    addEventListeners () {
      const guessCube = this.$refs.guessCube as HTMLElement
      guessCube.addEventListener('mousedown', this.onMouseDown)
      guessCube.addEventListener('mouseup', this.onMouseUp)
      guessCube.addEventListener('mouseenter', this.onMouseOver)
      guessCube.addEventListener('mouseleave', this.onMouseLeave)
      guessCube.addEventListener('dragenter', this.preventDefault)
      guessCube.addEventListener('dragover', this.preventDefault)
      guessCube.addEventListener('drop', this.onDrop)
    },
    removeEventListeners () {
      const guessCube = this.$refs.guessCube as HTMLElement
      guessCube.removeEventListener('mousedown', this.onMouseDown)
      guessCube.removeEventListener('mouseup', this.onMouseUp)
      guessCube.removeEventListener('mouseenter', this.onMouseOver)
      guessCube.removeEventListener('mouseleave', this.onMouseLeave)
      guessCube.removeEventListener('dragenter', this.preventDefault)
      guessCube.removeEventListener('dragover', this.preventDefault)
      guessCube.removeEventListener('drop', this.onDrop)
    }
  },
  mounted () {
    this.updateSelectedLetter(this.letters[0])
    this.addEventListeners()
  },
  beforeDestroy () {
    this.removeEventListeners()
  }
})
</script>

<style lang="scss" scoped>

$cubeSize: 100px;
$fontSize: 40px;
$translateZ: $cubeSize / 2;

.guess-cube {
  position: relative;
}

.cube-number {
  position: absolute;
  width: 100%;
  display: flex;
  justify-content: center;
  top: $cubeSize + 40px;
  z-index: -1;
}
.scene {
  width: $cubeSize;
  height: $cubeSize;
  perspective: $cubeSize * 2;
}

.cube {
  width: $cubeSize;
  height: $cubeSize;
  position: relative;
  transform-style: preserve-3d;
  transform: translateZ(-$translateZ);
  transition: transform 1s;
}

.cube.show-front  { transform: translateZ(-$translateZ) rotateY(   0deg); }
.cube.show-right  { transform: translateZ(-$translateZ) rotateY( -90deg); }
.cube.show-back   { transform: translateZ(-$translateZ) rotateY(-180deg); }
.cube.show-left   { transform: translateZ(-$translateZ) rotateY(  90deg); }
.cube.show-top    { transform: translateZ(-$translateZ) rotateX( -90deg); }
.cube.show-bottom { transform: translateZ(-$translateZ) rotateX(  90deg); }

.cube__face {
  position: absolute;
  width: $cubeSize;
  height: $cubeSize;
  border: 2px solid black;
  line-height: $cubeSize;
  font-size: $fontSize;
  font-weight: bold;
  color: black;
  text-align: center;
}

.cube__face  { background: hsla(  325, 100%, 50%, 1); }

.cube__face--front  { transform: rotateY(  0deg) translateZ($translateZ); }
.cube__face--right  { transform: rotateY( 90deg) translateZ($translateZ); }
.cube__face--back   { transform: rotateY(180deg) translateZ($translateZ); }
.cube__face--left   { transform: rotateY(-90deg) translateZ($translateZ); }
.cube__face--top    { transform: rotateX( 90deg) translateZ($translateZ); }
.cube__face--bottom { transform: rotateX(-90deg) translateZ($translateZ); }

.letters-hint {
  border: 1px solid black;
  background-color: white;
  color: black;
  padding: 3px 8px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-left: 3px;
  transition: opacity 0.6s;
  z-index: 9999;
  &.top {
    margin-bottom: 3px;
  }
  &.bottom {
    margin-top: 8px;
  }
}
</style>
