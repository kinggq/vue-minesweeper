<script setup lang="ts">
import { isDev, toggleDev } from '~/composables'
import { GamePlay } from '~/composables/logic'

const play = new GamePlay(6, 6, 3)
useStorage('vue-sweeper', play.state)
const state = computed(() => play.board)

const mineCount = computed(() => {
  return play.blocks.reduce((a, b) => a + (b.mine ? 1 : 0), 0)
})

watchEffect(() => {
  play.checkGameState()
})
</script>

<template>
  <div>
    <div>Minesweeper</div>

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
    <div>count: {{ mineCount }}</div>
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
