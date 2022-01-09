<template>
  <div id="game" class="game">
    <div class="cube-area">
      <div class="cube-row" id="cubeRow1" v-if="cubes.length >= 5">
        <GuessCube v-for="index in 5"
                   :key="index"
                   @letterUpdated="updateCubeLetter"
                   :letters="cubes[index - 1].letterSet"
                   :number="cubes[index - 1].number" />
      </div>
      <div class="cube-row" id="cubeRow2" v-if="cubes.length === 10">
        <GuessCube v-for="index in 5"
                   :key="index"
                   @letterUpdated="updateCubeLetter"
                   :letters="cubes[index + 4].letterSet"
                   :number="cubes[index + 4].number" />
      </div>
    </div>
    <div class="letter-slots">
      <div v-for="index in 6"
           :key="index" class="letter-slot"
           :id="'letterSlot' + index"></div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue'
import GuessCube from './GuessCube.vue'

interface Cube {
  number: number;
  selectedLetter: string;
  letterSet: string[];
  letterIndex: number|undefined;
}
export default Vue.extend({
  components: { GuessCube },
  props: {
    wordLength: {
      type: Number,
      required: true
    },
    cubeValues: {
      type: Array as PropType<string[][]>,
      required: true
    }
  },
  data: () => {
    return {
      draggedEl: null as HTMLElement | null,
      cubes: [] as Cube[]
    }
  },
  computed: {
    wordArray (): string[] {
      const arr: string[] = []
      for (let i = 0; i < this.wordLength; i++) {
        const cube = this.cubes.find(c => c.letterIndex === i)
        arr.push(cube ? cube.selectedLetter : '')
      }
      return arr
    }
  },
  methods: {
    updateCubeLetter (cubeNumber: number, cubeLetter: string) {
      this.cubes = this.cubes.map(c => {
        if (c.number === cubeNumber) {
          this.$set(c, 'selectedLetter', cubeLetter)
        }
        return c
      })
    },
    getIndexNumberFromId (el: HTMLElement): number|null {
      if (!el.id) return null
      const matches = el.id.match(/\d+/)
      if (!matches) return null

      return parseInt(matches[0])
    },
    draggedCubeDropped (dropEl: HTMLElement): void {
      if (!this.draggedEl) return
      if (dropEl.classList.contains('letter-slot')) {
        return this.draggedCubeDroppedIntoLetterSpace(dropEl)
      }
    },
    findLetterSlotParent (targetEl: HTMLElement): HTMLElement|null {
      if (targetEl.id.startsWith('letterSlot')) return targetEl
      let parentEl = targetEl.parentElement
      while (parentEl && parentEl.id !== 'game') {
        if (parentEl.id.startsWith('letterSlot')) return parentEl
        parentEl = parentEl.parentElement
      }
      return null
    },
    moveCubeElBackToArea (cubeEl: HTMLElement): void {
      const cubeNumber = this.getIndexNumberFromId(cubeEl)
      if (!cubeNumber) return
      const rowId = cubeNumber <= 5 ? 1 : 2
      const rowEl = document.getElementById('cubeRow' + rowId)
      if (!rowEl) return
      for (let c = 0; c < rowEl.children.length; c++) {
        const child = rowEl.children[c] as HTMLElement
        const childIndex = this.getIndexNumberFromId(child)
        if (!childIndex) continue
        if (cubeNumber === childIndex - 1) {
          rowEl.insertBefore(cubeEl, child)
          break
        }
      }
    },
    draggedCubeDroppedIntoLetterSpace (letterSpaceEl: HTMLElement): void {
      if (!this.draggedEl) return
      let letterIndex = this.getIndexNumberFromId(letterSpaceEl)
      if (!letterIndex) return console.error('no letter index')
      letterIndex -= 1
      const cubeNumber = this.getIndexNumberFromId(this.draggedEl)
      if (this.cubes.some(c => c.number === cubeNumber && c.letterIndex === letterIndex)) {
        // already there. do nothing
        return
      }
      this.cubes = this.cubes.map(c => {
        if (c.letterIndex === letterIndex) {
          const replacedCubeEl = document.getElementById('guessCube' + c.number) as HTMLElement
          this.moveCubeElBackToArea(replacedCubeEl)
          this.$set(c, 'letterIndex', undefined)
          return c
        }
        if (c.number === cubeNumber) {
          this.$set(c, 'letterIndex', letterIndex)
        }
        return c
      });
      (this.draggedEl.parentNode as HTMLElement).removeChild(this.draggedEl)
      letterSpaceEl.appendChild(this.draggedEl)
      this.draggedEl = null
    },
    onDrop (event: DragEvent): void {
      console.log('drop', event)
      if (!this.draggedEl) return
      const targetEl = event.target as HTMLElement
      const letterSpaceEl = this.findLetterSlotParent(targetEl)
      if (letterSpaceEl) {
        return this.draggedCubeDroppedIntoLetterSpace(letterSpaceEl)
      }
      this.moveCubeElBackToArea(this.draggedEl)
      this.draggedEl = null
    },
    preventDefault (event: DragEvent): void {
      event.preventDefault()
    },
    onDragStart (event: DragEvent): void {
      if (event.target) {
        const el = event.target as HTMLElement
        if (el.id.startsWith('guessCube')) {
          this.draggedEl = el
        }
      }
    },
    addEventListeners (): void {
      document.addEventListener('dragstart', this.onDragStart)
      document.addEventListener('dragenter', this.preventDefault)
      document.addEventListener('dragover', this.preventDefault)
      document.addEventListener('drop', this.onDrop)
    },
    removeEventListeners (): void {
      document.removeEventListener('dragstart', this.onDragStart)
      document.removeEventListener('dragenter', this.preventDefault)
      document.removeEventListener('dragover', this.preventDefault)
      document.removeEventListener('drop', this.onDrop)
    }
  },
  mounted () {
    for (let s = 0; s < this.cubeValues.length; s++) {
      this.cubes.push({
        number: s + 1,
        selectedLetter: this.cubeValues[s][0],
        letterSet: this.cubeValues[s],
        letterIndex: undefined
      })
    }
    this.addEventListeners()
  },
  beforeDestroy () {
    this.removeEventListeners()
  }
})
</script>

<style scoped>
  .game {
    margin-top: 50px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
  .letter-slots {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 90%;
    margin-top: 100px;
  }
  .letter-slot {
    width: 120px;
    height: 160px;
    margin: 0 25px;
    border: 5px solid black;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .cube-area {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 100%;
    min-height: 300px;
  }

  .cube-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 75%;
    margin-top: 50px;
  }
</style>
