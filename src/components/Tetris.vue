<template>
  <div class="tetris">
    <h3 class="tetris-score">Score: <span>0</span></h3>
    <button class="tetris-start-btn">Start / Pause</button>
    <div class="tetris-grid" ref="grid">
      <div v-for="i in 200" :key="i">{{ i }}</div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue'

export default defineComponent({
  name: 'Tetris',

  setup() {
    const width = 10
    const height = 20
    const grid = ref(null)

    return {
      grid
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

  .tetris-score {
    margin-bottom: 20px;
  }

  .tetris-start-btn {
    margin-bottom: 20px;
  }

  .tetris-grid {
    background: blue;
    height: 600px;
    width: 300px;
    display: grid;
    grid-template-columns: repeat(10, 30px);

    div {
      background: yellow;
      width: 30px;
      height: 30px;
    }
  }
}
</style>
