<template>
  <div class="tetris">
    <canvas id="tetris-canvas" ref="canvas"></canvas>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref } from 'vue'

class coordinate {
  x: number
  y: number
  constructor(x: number, y: number) {
    this.x = x
    this.y = y
  }
}

let DIRECTION = {
  IDLE: 0,
  DOWN: 1,
  LEFT: 2,
  RIGHT: 3
}

export default defineComponent({
  name: 'Tetris',

  setup() {
    const canvas = ref(null)
    let ctx: CanvasRenderingContext2D
    let canvasHeight = 20
    let canvasWidth = 12
    let startX = 4
    let startY = 0
    // Makes a 2d array with all possible coordinates in pixels
    let coordArray = [...Array(canvasHeight)].map(() =>
      Array(canvasWidth).fill(0)
    )
    let currTetromino = [
      [1, 0], // x = 1, y = 0
      [0, 1], // x = 0, y = 1
      [1, 1],
      [2, 1]
    ]

    // Array that holds all possible tetrominos
    let tetrominos = []

    // Array that holds all possible colors for the tetrominos
    let tetrominosColors = [
      'purple',
      'cyan',
      'blue',
      'yellow',
      'orange',
      'green',
      'red'
    ]
    let currTetrominoColor: string

    let tetrisArray = [...Array(canvasHeight)].map(() =>
      Array(canvasWidth).fill(0)
    )

    let direction

    const populateCoordArray = () => {
      let i = 0
      let j = 0
      let increment = 23
      // start at y = 9 = 9px
      // until 446 pixels (height of canvas)
      // +23 pixels each time (1 box = 21px)
      for (let y = 9; y <= 446; y += increment) {
        // start at x = 11 = 11px
        // until 264 pixels (width of canvas)
        for (let x = 11; x <= 264; x += increment) {
          coordArray[i][j] = new coordinate(x, y)
          i++
        }
        j++
        i = 0
      }

      // console.log(coordArray)
    }

    const drawTetromino = () => {
      for (let i = 0; i < currTetromino.length; i++) {
        let x = currTetromino[i][0] + startX // new tetromino starts 4 boxes from the right
        let y = currTetromino[i][1] + startY // new tetromino starts 0 boxes from the top
        tetrisArray[x][y] = 1
        let coordX = coordArray[x][y].x // gets x value from the Coordinate class
        let coordY = coordArray[x][y].y // gets y value from the Coordinate class
        ctx.fillStyle = currTetrominoColor
        ctx.fillRect(coordX, coordY, 21, 21)
      }
    }

    const setupCanvas = () => {
      ctx = ((canvas.value as unknown) as HTMLCanvasElement).getContext('2d')!
      ;((canvas.value as unknown) as HTMLCanvasElement).width = 936
      ;((canvas.value as unknown) as HTMLCanvasElement).height = 956

      ctx.scale(2, 2)
      ctx.fillStyle = 'white'
      ctx.fillRect(
        0,
        0,
        ((canvas.value as unknown) as HTMLCanvasElement).width,
        ((canvas.value as unknown) as HTMLCanvasElement).height
      )
      ctx.strokeStyle = 'black'
      ctx.strokeRect(8, 8, 280, 462)

      populateCoordArray()
      drawTetromino()
    }

    onMounted(() => {
      setupCanvas()
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
