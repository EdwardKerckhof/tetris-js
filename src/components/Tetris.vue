<template>
  <div class="tetris">
    <canvas id="tetris-canvas" ref="canvas"></canvas>
  </div>
</template>

<script>
import { defineComponent, onMounted, ref } from 'vue'

export default defineComponent({
  name: 'Tetris',

  setup() {
    const canvas = ref(null)
    let ctx
    let canvasHeight = 20
    let canvasWidth = 12
    let startX = 4
    let startY = 0
    // Makes a 2d array with all possible coordinates in pixels
    let coordArray = [...Array(canvasHeight)].map(() =>
      Array(canvasWidth).fill(0)
    )

    onMounted(() => {
      console.log(coordArray)
    })

    return {
      canvas
    }
  }
})
</script>

<style lang="scss" scoped>
.tetris {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: left;

  #tetris-canvas {
    background: red;
  }
}
</style>
