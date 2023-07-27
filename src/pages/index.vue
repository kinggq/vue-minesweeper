<script setup lang="ts">
import { isDev, toggleDev } from '~/composables'
import { GamePlay } from '~/composables/logic'

const play = new GamePlay(9, 9, 10)

const now = useNow()
const timerMS = computed(() => Math.round((+now.value - +play.state.value.startMs) / 1000))
useStorage('vue-sweeper', play.state)
const state = computed(() => play.board)

const mineCount = computed(() => {
  if (!play.state.value.mineGenerated)
    return play.mines
  return play.blocks.reduce((a, b) => a + (b.mine ? 1 : 0) - (b.flagged ? 1 : 0), 0)
})

function newGame(difficulty: 'easy' | 'medium' | 'hard') {
  switch (difficulty) {
    case 'easy':
      play.reset(9, 9, 10)
      break
    case 'medium':
      play.reset(16, 16, 40)
      break
    case 'hard':
      play.reset(16, 30, 99)
      break
  }
}

watchEffect(() => {
  play.checkGameState()
})
</script>

<template>
  <div>
    <div>Minesweeper</div>
    <div flex="~ gap1" justify-center p4>
      <button btn @click="play.reset()">
        New Game
      </button>
      <button btn @click="newGame('easy')">
        Easy
      </button>
      <button btn @click="newGame('medium')">
        Medium
      </button>
      <button btn @click="newGame('hard')">
        Hard
      </button>
    </div>
    <div flex="~ gap10" justify-center>
      <div flex="~ gap1" items-center font-mono text-2xl>
        <div i-carbon-timer />
        {{ timerMS }}
      </div>
      <div flex="~ gap1" items-center font-mono text-2xl>
        <div i-mdi-mine />
        {{ mineCount }}
      </div>
    </div>
    <div w-full overflow-auto p5>
      <div
        v-for="(row, y) in state" :key="y"
        ma
        w-max
        items-center
        justify-center flex="~"
      >
        <MineBlock
          v-for="(block, x) in row" :key="x"
          :block="block"
          @click="play.onClick(block)"
          @contextmenu.prevent="play.onRightClick(block)"
        />
      </div>
    </div>

    <!-- <div>count: {{ mineCount }}</div> -->
    <div flex="~ gap-1" justify-center>
      <button btn @click="toggleDev()">
        {{ isDev ? 'DEV' : 'NORMAL' }}
      </button>
      <button btn @click="play.reset">
        RESET
      </button>
    </div>
  </div>
  <Confetti :passed="play.state.value.gameState === 'won'" />
</template>
