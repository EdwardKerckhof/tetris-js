<template>
  <div class="tetris">
    <div class="tetris-actions">
      <button>Start / Pause</button>
      <h3>
        Score
        <p>{{ score }}</p>
      </h3>
      <h3>
        Level
        <p>{{ level }}</p>
      </h3>
      <h3>
        State
        <p>
          {{ state === 0 ? 'Playing' : state === 1 ? 'Game Over' : 'Paused' }}
        </p>
      </h3>
      <h3>
        Controls
        <p class="control">ArrowDown: Move Down</p>
        <p class="control">ArrowRight: Move Right</p>
        <p class="control">ArrowLeft: Move Left</p>
        <p class="control">ArrowUp: Rotate</p>
      </h3>
    </div>
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

const DIRECTION = {
  IDLE: 0,
  DOWN: 1,
  LEFT: 2,
  RIGHT: 3
}

const STATE = {
  PLAYING: 0,
  GAME_OVER: 1,
  PAUSED: 2
}

export default defineComponent({
  name: 'Tetris',

  setup() {
    const canvas = ref(null)
    let ctx: CanvasRenderingContext2D
    let arrayHeight = 20
    let arrayWidth = 12
    let startX = 4
    let startY = 0
    let score = ref(0)
    let level = ref(1)
    let state = ref(STATE.PAUSED)
    // Makes a 2d array with all possible coordinates and fills it with zeros (12x20)
    let coordArray = [...Array(arrayHeight)].map(() =>
      Array(arrayWidth).fill(0)
    )
    let currTetromino = [
      [1, 0], // x = 1, y = 0
      [0, 1], // x = 0, y = 1
      [1, 1],
      [2, 1]
    ]

    // Array that holds all possible tetrominos
    let tetrominos: Array<number[][]> = []

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

    // Makes a 2d array of the board and fills it with zeros (12x20)
    let tetrisArray = [...Array(arrayHeight)].map(() =>
      Array(arrayWidth).fill(0)
    )

    // Makes a 2d array of the board and fills the already placed tetrominos (12x20)
    let stoppedShapesArray = [...Array(arrayHeight)].map(() =>
      Array(arrayWidth).fill(0)
    )

    let direction: number

    const populateCoordArray = () => {
      let i = 0
      let j = 0
      let increment = 23
      // start at y = 9 = 9px
      // until 446 pixels (height of canvas)
      // +23 pixels each time (1 box = 21x21px)
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
        tetrisArray[x][y] = 1 // 1 = fill
        let coordX = coordArray[x][y].x // gets x value from the Coordinate class
        let coordY = coordArray[x][y].y // gets y value from the Coordinate class
        ctx.fillStyle = currTetrominoColor
        ctx.fillRect(coordX, coordY, 21, 21)
      }
    }

    const deleteTetromino = () => {
      for (let i = 0; i < currTetromino.length; i++) {
        let x = currTetromino[i][0] + startX // old tetromino x
        let y = currTetromino[i][1] + startY // old tetromino y
        tetrisArray[x][y] = 0 // 0 = empty
        let coordX = coordArray[x][y].x // gets x value from the Coordinate class
        let coordY = coordArray[x][y].y // gets y value from the Coordinate class
        ctx.fillStyle = 'white'
        ctx.fillRect(coordX, coordY, 21, 21)
      }
    }

    const handleKeyDown = (e: KeyboardEvent) => {
      switch (state.value) {
        case STATE.PLAYING:
          switch (e.key) {
            case 'ArrowLeft':
              direction = DIRECTION.LEFT
              if (!hitsWall() && !horizontalCollision()) {
                deleteTetromino()
                startX--
                drawTetromino()
              }
              break
            case 'ArrowRight':
              direction = DIRECTION.RIGHT
              if (!hitsWall() && !horizontalCollision()) {
                deleteTetromino()
                startX++
                drawTetromino()
              }
              break
            case 'ArrowDown':
              handleTetrominoArrowDown()
              break
            case 'ArrowUp':
              handleTetrominoRotation()
              break
          }
          break
      }
    }

    const handleTetrominoRotation = () => {}

    const handleTetrominoArrowDown = () => {
      direction = DIRECTION.DOWN
      if (!verticalCollision()) {
        deleteTetromino()
        startY++
        drawTetromino()
      }
    }

    const hitsWall = () => {
      // check if ((x <= 0) || (x >= 11) && moving left or right) -> stop movement
      for (let i = 0; i < currTetromino.length; i++) {
        let x = currTetromino[i][0] + startX
        // console.log(x)
        if (x <= 0 && direction === DIRECTION.LEFT) return true
        else if (x >= 11 && direction === DIRECTION.RIGHT) return true
        return false
      }
    }

    const checkCompletedRows = () => {
      let rowsToDelete = 0
      let startOfDeletion = 0
      for (let y = 0; y < arrayHeight; y++) {
        let completed = true
        for (let x = 0; x < arrayWidth; x++) {
          let square = stoppedShapesArray[x][y]
          // square = 0 (nothing there) of it's of type undefined (so not string) -> not completed
          if (square === 0 || typeof square === 'undefined') {
            completed = false
            break
          }
        }

        if (completed) {
          if (startOfDeletion === 0) startOfDeletion = y
          rowsToDelete++

          for (let i = 0; i < arrayWidth; i++) {
            // set values in stopped and current board array (tetrisArray) to 0, meaning nothing is there
            stoppedShapesArray[i][y] = 0
            tetrisArray[i][y] = 0
            let coordX = coordArray[i][y].x
            let coordY = coordArray[i][y].y
            // color squares to delete white
            ctx.fillStyle = 'white'
            ctx.fillRect(coordX, coordY, 21, 21)
          }
        }
      }

      // if a row needs to be deleted add 10 to score
      if (rowsToDelete > 0) {
        score.value += 10
        moveAllRowsDown(rowsToDelete, startOfDeletion)
      }
    }

    const moveAllRowsDown = (rows: number, start: number) => {
      // move from start to 0
      for (let i = start - 1; i >= 0; i--) {
        for (let x = 0; x < arrayWidth; x++) {
          let y2 = i + rows
          let square = stoppedShapesArray[x][i]
          let nextSquare = stoppedShapesArray[x][y2]
          if (typeof square === 'string') {
            nextSquare = square
            tetrisArray[x][y2] = 1
            stoppedShapesArray[x][y2] = square
            let coordX = coordArray[x][y2].x
            let coordY = coordArray[x][y2].y
            // color squares to delete white
            ctx.fillStyle = nextSquare
            ctx.fillRect(coordX, coordY, 21, 21)

            square = 0
            tetrisArray[x][i] = 0
            stoppedShapesArray[x][i] = 0
            coordX = coordArray[x][i].x
            coordY = coordArray[x][i].y
            // color squares to delete white
            ctx.fillStyle = 'white'
            ctx.fillRect(coordX, coordY, 21, 21)
          }
        }
      }
    }

    const newTetrominoSpawn = () => {
      createTetromino()
      direction = DIRECTION.IDLE
      startX = 4
      startY = 0
      drawTetromino()
    }

    const verticalCollision = () => {
      let tetromino = currTetromino
      let collision = false
      for (let i = 0; i < tetromino.length; i++) {
        let square = tetromino[i]
        let x = square[0] + startX
        let y = square[1] + startY

        if (direction === DIRECTION.DOWN) y++
        if (tetrisArray[x][y + 1] === 1) {
          // if there is a string in the next down square it means it has a color so there is already a square in place
          if (typeof stoppedShapesArray[x][y + 1] === 'string') {
            deleteTetromino()
            startY++
            drawTetromino()
            collision = true
            break
          }
          // bottom
          if (y >= 20) {
            collision = true
            break
          }
        }

        if (collision) {
          // if startY of new tetromino <= 2 -> game over
          if (startY <= 2) {
            state.value = STATE.GAME_OVER
            ctx.fillStyle = 'white'
          } else {
            for (let i = 0; i < tetromino.length; i++) {
              let square = tetromino[i]
              let x = square[0] + startX
              let y = square[1] + startY
              stoppedShapesArray[x][y] = currTetrominoColor
            }
            checkCompletedRows()
            newTetrominoSpawn()
          }
        }
      }
      return collision
    }

    const horizontalCollision = () => {
      let tetromino = currTetromino
      for (let i = 0; i < tetromino.length; i++) {
        let square = tetromino[i]
        let x = square[0] + startX
        let y = square[1] + startY

        if (direction === DIRECTION.LEFT) x--
        else if (direction === DIRECTION.RIGHT) x++
        // if there is a string in the next down square it means it has a color so there is already a square in place
        if (typeof stoppedShapesArray[x][y] === 'string') {
          return true
        }
      }
      return false
    }

    // creates all tetromino shapes
    const createTetrominos = () => {
      // I
      tetrominos.push([
        [0, 0],
        [1, 0],
        [2, 0],
        [3, 0]
      ])
      // J
      tetrominos.push([
        [0, 0],
        [0, 1],
        [1, 1],
        [2, 1]
      ])
      // T
      tetrominos.push([
        [1, 0],
        [0, 1],
        [1, 1],
        [2, 1]
      ])
      // L
      tetrominos.push([
        [2, 0],
        [0, 1],
        [1, 1],
        [2, 1]
      ])
      // S
      tetrominos.push([
        [1, 0],
        [2, 0],
        [0, 1],
        [1, 1]
      ])
      // Z
      tetrominos.push([
        [0, 0],
        [1, 0],
        [1, 1],
        [2, 1]
      ])
      // O or square
      tetrominos.push([
        [0, 0],
        [1, 0],
        [0, 1],
        [1, 1]
      ])
    }

    const createTetromino = () => {
      let randomTetroIndex = Math.floor(Math.random() * tetrominos.length)
      currTetromino = tetrominos[randomTetroIndex]
      currTetrominoColor = tetrominosColors[randomTetroIndex]
    }

    const setupCanvas = () => {
      const scale = 2
      ctx = ((canvas.value as unknown) as HTMLCanvasElement).getContext('2d')!
      ;((canvas.value as unknown) as HTMLCanvasElement).width = 296 * scale
      ;((canvas.value as unknown) as HTMLCanvasElement).height = 478 * scale

      ctx.scale(scale, scale)
      ctx.fillStyle = 'white'
      ctx.fillRect(
        0,
        0,
        ((canvas.value as unknown) as HTMLCanvasElement).width,
        ((canvas.value as unknown) as HTMLCanvasElement).height
      )
      ctx.strokeStyle = 'black'
      ctx.strokeRect(8, 8, 280, 462)

      document.addEventListener('keydown', handleKeyDown)
      createTetrominos()
      createTetromino()

      populateCoordArray()
      drawTetromino()
    }

    onMounted(() => {
      setupCanvas()
    })

    return {
      score,
      level,
      state,
      canvas
    }
  }
})
</script>

<style lang="scss" scoped>
.tetris {
  display: flex;
  flex-direction: row-reverse;
  justify-content: center;
  text-align: left;

  canvas {
    margin-right: 50px;
  }

  .tetris-actions {
    button {
      background: none;
      border: none;
      font-size: 16px;
      text-transform: uppercase;
      border: 2px solid #2c3e50;
      color: #2c3e50;
      padding: 5px 10px;
      border-radius: 5px;
      font-weight: bold;
      cursor: pointer;
      outline: none;
      margin-top: 10px;
      transition: all 0.15s ease;

      &:hover {
        box-shadow: 0px 2px 5px rgba($color: #000000, $alpha: 0.15);
      }
    }

    h3 {
      margin: 20px 0;

      p {
        font-size: 16px;
        padding: 5px 12px;
        border: 2px solid #2c3e50;

        &.control {
          border: none;
          border-right: 2px solid;
          border-left: 2px solid;
          text-transform: none;

          &:nth-child(1) {
            border-top: 2px solid;
          }

          &:nth-child(4) {
            border-bottom: 2px solid;
          }
        }
      }
    }
  }
}
</style>
