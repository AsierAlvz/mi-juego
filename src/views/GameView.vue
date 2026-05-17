<template>
  <div class="game-container">
    <h1 class="title">⚔️ Elden Ring Memory</h1>
    <p class="subtitle">Encuentra todas las parejas del Mundo Entre</p>

    <div class="info-bar">
      <div class="stat">
        <span class="stat-label">Movimientos</span>
        <span class="stat-value">{{ moves }}</span>
      </div>
      <div class="stat">
        <span class="stat-label">Parejas</span>
        <span class="stat-value">{{ pairsFound }}/{{ totalPairs }}</span>
      </div>
      <div class="stat">
        <span class="stat-label">Tiempo</span>
        <span class="stat-value">{{ formattedTime }}</span>
      </div>
    </div>

    <div class="board">
      <div
        v-for="(card, index) in cards"
        :key="card.id"
        class="card"
        :class="{
          flipped: card.flipped,
          matched: card.matched,
        }"
        @click="flipCard(index)"
      >
        <div class="card-inner">
          <div class="card-back">🔥</div>
          <div class="card-front">
            <span class="card-emoji">{{ card.emoji }}</span>
            <span class="card-name">{{ card.name }}</span>
          </div>
        </div>
      </div>
    </div>

    <div class="buttons">
      <button class="btn" @click="resetGame">🔄 Nueva partida</button>
      <button class="btn" @click="changeDifficulty">
        Dificultad: {{ difficulty === 'easy' ? 'Fácil' : 'Difícil' }}
      </button>
    </div>

    <div v-if="gameWon" class="win-overlay">
      <div class="win-box">
        <h2>⚔️ ¡Elden Lord!</h2>
        <p>Completado en {{ moves }} movimientos y {{ formattedTime }}</p>
        <p class="record" v-if="isNewRecord">🏆 ¡Nuevo récord!</p>
        <button class="btn" @click="resetGame">Jugar de nuevo</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const allCards = [
  { emoji: '🐉', name: 'Rennala' },
  { emoji: '⚔️', name: 'Margit' },
  { emoji: '🌙', name: 'Ranni' },
  { emoji: '🔥', name: 'Maliketh' },
  { emoji: '👁️', name: 'Mohg' },
  { emoji: '🗡️', name: 'Malenia' },
  { emoji: '🌑', name: 'Radahn' },
  { emoji: '💀', name: 'Godrick' },
]

const difficulty = ref('easy')
const cards = ref([])
const flippedCards = ref([])
const moves = ref(0)
const pairsFound = ref(0)
const time = ref(0)
const gameWon = ref(false)
const bestScore = ref(
  localStorage.getItem('eldenBest') ? parseInt(localStorage.getItem('eldenBest')) : null,
)
const isNewRecord = ref(false)
let timer = null

const totalPairs = computed(() => (difficulty.value === 'easy' ? 6 : 8))

const formattedTime = computed(() => {
  const m = Math.floor(time.value / 60)
    .toString()
    .padStart(2, '0')
  const s = (time.value % 60).toString().padStart(2, '0')
  return `${m}:${s}`
})

function generateCards() {
  const selected = allCards.slice(0, totalPairs.value)
  const doubled = [...selected, ...selected].map((card, i) => ({
    ...card,
    id: i,
    flipped: false,
    matched: false,
  }))
  return doubled.sort(() => Math.random() - 0.5)
}

function resetGame() {
  cards.value = generateCards()
  flippedCards.value = []
  moves.value = 0
  pairsFound.value = 0
  time.value = 0
  gameWon.value = false
  isNewRecord.value = false
  clearInterval(timer)
  timer = setInterval(() => {
    time.value++
  }, 1000)
}

function flipCard(index) {
  const card = cards.value[index]
  if (card.flipped || card.matched || flippedCards.value.length === 2) return

  card.flipped = true
  flippedCards.value.push(index)

  if (flippedCards.value.length === 2) {
    moves.value++
    checkMatch()
  }
}

function checkMatch() {
  const [i1, i2] = flippedCards.value
  const c1 = cards.value[i1]
  const c2 = cards.value[i2]

  if (c1.name === c2.name) {
    c1.matched = true
    c2.matched = true
    pairsFound.value++
    flippedCards.value = []
    if (pairsFound.value === totalPairs.value) endGame()
  } else {
    setTimeout(() => {
      c1.flipped = false
      c2.flipped = false
      flippedCards.value = []
    }, 1000)
  }
}

function endGame() {
  clearInterval(timer)
  gameWon.value = true
  if (!bestScore.value || moves.value < bestScore.value) {
    bestScore.value = moves.value
    localStorage.setItem('eldenBest', moves.value)
    isNewRecord.value = true
  }
}

function changeDifficulty() {
  difficulty.value = difficulty.value === 'easy' ? 'hard' : 'easy'
  resetGame()
}

onMounted(() => resetGame())
onUnmounted(() => clearInterval(timer))
</script>

<style scoped>
.game-container {
  min-height: 100vh;
  padding: 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  background: radial-gradient(ellipse at top, #1a0a00 0%, #0a0a0a 70%);
}

.title {
  font-size: 2.5rem;
  color: #c9a84c;
  text-shadow: 0 0 20px #c9a84c88;
  margin-bottom: 0.5rem;
}

.subtitle {
  color: #888;
  margin-bottom: 2rem;
  font-style: italic;
}

.info-bar {
  display: flex;
  gap: 2rem;
  margin-bottom: 2rem;
  background: #1a1a1a;
  padding: 1rem 2rem;
  border-radius: 8px;
  border: 1px solid #c9a84c44;
}

.stat {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}

.stat-label {
  font-size: 0.75rem;
  color: #888;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.stat-value {
  font-size: 1.5rem;
  color: #c9a84c;
  font-weight: bold;
}

.board {
  display: grid;
  grid-template-columns: repeat(4, 120px);
  gap: 1rem;
  margin-bottom: 2rem;
}

.card {
  width: 120px;
  height: 120px;
  cursor: pointer;
  perspective: 1000px;
}

.card-inner {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.5s;
}

.card.flipped .card-inner,
.card.matched .card-inner {
  transform: rotateY(180deg);
}

.card-back,
.card-front {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border: 2px solid #c9a84c44;
}

.card-back {
  background: #1a1a1a;
  font-size: 2.5rem;
}

.card-front {
  background: #2a1a00;
  transform: rotateY(180deg);
  border-color: #c9a84c;
  gap: 8px;
}

.card.matched .card-front {
  background: #1a2a00;
  border-color: #4caf50;
}

.card-emoji {
  font-size: 2rem;
}

.card-name {
  font-size: 0.7rem;
  color: #c9a84c;
  text-align: center;
}

.buttons {
  display: flex;
  gap: 1rem;
}

.btn {
  padding: 0.75rem 1.5rem;
  background: transparent;
  border: 1px solid #c9a84c;
  color: #c9a84c;
  border-radius: 4px;
  cursor: pointer;
  font-family: 'Georgia', serif;
  font-size: 0.9rem;
  transition: all 0.3s;
}

.btn:hover {
  background: #c9a84c22;
}

.win-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #000000cc;
  display: flex;
  align-items: center;
  justify-content: center;
}

.win-box {
  background: #1a1a1a;
  border: 2px solid #c9a84c;
  border-radius: 12px;
  padding: 2rem 3rem;
  text-align: center;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.win-box h2 {
  font-size: 2rem;
  color: #c9a84c;
}

.record {
  color: #ffd700;
  font-size: 1.2rem;
}
</style>
