<!-- src/components/GameGrid.vue -->
<template>
  <div class="game-container">
    <h1 class="header-title">Ethereal Chronicles: The Veil of Arcane Eternity</h1>
    <div>
      <img
        src="~/assets/game-title-logo.webp"
        alt="Game Logo"
        class="game-logo"
        style="width: 20rem"
      />
    </div>

    <div id="game" class="game-grid">
      <span v-for="(cell, cellIndex) in grid.flat()" :key="cellIndex" class="cell">{{
        cell
      }}</span>
    </div>
    <div class="game-log">
      <h3>ゲームログ</h3>
      <div class="log-area">
        <div class="player-status">
          <div class="player-emoji">😀</div>
          <div class="player-hp">HP: {{ player.hp }}/10</div>
        </div>
        <div class="log-content">
          <p v-for="(message, index) in gameLog" :key="index">{{ message }}</p>
        </div>
      </div>
    </div>
    <p>Controls: K (Up), H (Left), J (Down), L (Right)</p>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, computed } from 'vue'
// import { logger } from 'workbox-core/_private'

const VIEWPORT_SIZE = 15  // 表示するグリッドのサイズイズ
const GRID_SIZE = 50 + (VIEWPORT_SIZE * 2)  // マップサを拡大して壁の厚さを確保
const WALL = '🧱'  // 壁のエモジを追加
const ACTUAL_PLAY_AREA = 50  // 実際のプレイ可能エリア

class Player {
  x: number
  y: number
  hp: number

  constructor(x: number, y: number, hp: number = 10) {
    this.x = x
    this.y = y
    this.hp = hp
  }

  move(dx: number, dy: number) {
    this.x += dx
    this.y += dy
  }
}

class Enemy {
  x: number
  y: number

  constructor(x: number, y: number) {
    this.x = x
    this.y = y
  }
}

export default defineComponent({
  name: 'GameGrid',
  setup() {
    const grid = ref<string[][]>([])
    const player = new Player(0, 0)
    const enemies = ref<Enemy[]>([])
    const gameLog = ref<string[]>([])

    const generateEnemies = () => {
      const enemiesArray: Enemy[] = []
      for (let i = 0; i < 5; i++) {
        const x = Math.floor(Math.random() * ACTUAL_PLAY_AREA)
        const y = Math.floor(Math.random() * ACTUAL_PLAY_AREA)
        enemiesArray.push(new Enemy(x, y))
      }
      return enemiesArray
    }

    const initializeGrid = () => {
      // まず全てのセルを壁で初期化
      grid.value = Array.from({ length: GRID_SIZE }, () => Array(GRID_SIZE).fill(WALL))
      
      // プレイ可能エリアを作成（内側を空白に）
      for (let i = VIEWPORT_SIZE; i < VIEWPORT_SIZE + ACTUAL_PLAY_AREA; i++) {
        for (let j = VIEWPORT_SIZE; j < VIEWPORT_SIZE + ACTUAL_PLAY_AREA; j++) {
          grid.value[i]![j] = '⬛'
        }
      }

      // プレイヤーをランダムな位置に配置
      const randomX = Math.floor(Math.random() * ACTUAL_PLAY_AREA)
      const randomY = Math.floor(Math.random() * ACTUAL_PLAY_AREA)
      player.x = randomX + VIEWPORT_SIZE
      player.y = randomY + VIEWPORT_SIZE
      grid.value[player.x]![player.y] = '😀'

      enemies.value = generateEnemies()
      for (const enemy of enemies.value) {
        const x = enemy.x + VIEWPORT_SIZE
        const y = enemy.y + VIEWPORT_SIZE
        if (grid.value[x]![y] === '⬛') {
          grid.value[x]![y] = '👾'
        }
      }
    }

    const displayGrid = computed(() => {
      console.log('displayGrid')

      return grid.value.map((row) => row.join(' ')).join('\n')
    })

    const viewportGrid = computed(() => {
      // プレイヤーを常に中心に表示
      const startX = player.x - Math.floor(VIEWPORT_SIZE / 2)
      const startY = player.y - Math.floor(VIEWPORT_SIZE / 2)
      
      const viewport: string[][] = []
      for (let i = 0; i < VIEWPORT_SIZE; i++) {
        viewport[i] = []
        for (let j = 0; j < VIEWPORT_SIZE; j++) {
          const gridX = startX + i
          const gridY = startY + j
          viewport[i]![j] = grid.value[gridX]?.[gridY] ?? WALL
        }
      }
      return viewport
    })

    const addLogMessage = (message: string) => {
      gameLog.value.push(message)
      if (gameLog.value.length > 5) {
        gameLog.value.shift()
      }
    }

    const movePlayer = (dx: number, dy: number) => {
      const newX = player.x + dx
      const newY = player.y + dy

      if (grid.value[newX]?.[newY] === WALL) {
        addLogMessage('壁にぶつかりました！')
        return
      }

      grid.value[player.x]![player.y] = '⬛'
      player.move(dx, dy)
      
      if (grid.value[player.x]![player.y] === '👾') {
        addLogMessage('敵と遭遇！ダメージを受けました。')
        player.hp -= 1
        if (player.hp <= 0) {
          addLogMessage('ゲームオーバー！')
          alert('Game Over!')
          window.location.reload()
        }
      }

      grid.value[player.x]![player.y] = '😀'
    }

    const setupControls = () => {
      window.addEventListener('keydown', (event) => {
        switch (event.key) {
          case 'k':
            movePlayer(-1, 0)
            break // Up
          case 'j':
            movePlayer(1, 0)
            break // Down
          case 'h':
            movePlayer(0, -1)
            break // Left
          case 'l':
            movePlayer(0, 1)
            break // Right
        }
      })
    }

    onMounted(() => {
      initializeGrid()
      setupControls()
    })

    return {
      gridDisplay: displayGrid,
      grid: viewportGrid,
      gameLog,
      player,
    }
  },
})
</script>

<style scoped>
.game-container {
  text-align: center;
  justify-content: center;
  flex-direction: column;
}

.game-grid {
  background-color: #1d1d1d; /* ダーク背景色 */
  color: #e0e0e0; /* 明るいテキスト色 */
  border: 1px solid #333; /* ダークボーダー */
  padding: 20px;
  border-radius: 8px;
  display: grid; /* グリッドレイアウト */
  grid-template-columns: repeat(15, 2.5ch); /* 50から15に変更 */
  grid-template-rows: repeat(15, 2.5ch); /* 50から15に変更 */
  margin: 20px auto;
  font-family: monospace;
  white-space: pre;
  max-width: fit-content;
  margin-left: auto;
  margin-right: auto;
  /* line-height: 1ch; 縦幅を横幅に揃える */
}

/* .cell {
  width: 1ch;
  height: 1ch;
  text-align: center;
} */

.game-grid-item {
  background-color: #2c2c2c; /* アイテムのダーク背景色 */
  color: #e0e0e0; /* アイテムの明るいテキスト色 */
  border: 1px solid #444; /* アイテムのダークボーダー */
  padding: 10px;
  margin: 10px;
  border-radius: 4px;
}

.cell {
  display: flex;
  align-items: center;
  justify-content: center;
}

.game-log {
  background-color: #1d1d1d;
  border: 1px solid #333;
  border-radius: 8px;
  padding: 10px;
  margin: 20px auto;
  max-width: 40rem;
  text-align: left;
}

.log-area {
  display: flex;
  gap: 20px;
}

.player-status {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 100px;
}

.player-emoji {
  font-size: 3rem;
  margin-bottom: 10px;
}

.player-hp {
  color: #e0e0e0;
  font-size: 0.9rem;
}

.log-content {
  flex-grow: 1;
  height: 120px;
  overflow-y: auto;
  padding: 10px;
  border-left: 1px solid #333;
}

.log-content p {
  margin: 5px 0;
  color: #e0e0e0;
}
</style>
