<template>
  <div class="container">
    <h1>🎬 Movie Blasters</h1>

    <div v-if="!started">
      <button @click="startGame" :disabled="loading">
        {{ loading ? 'Loading movies...' : 'Start Game' }}
      </button>
    </div>

    <div v-else>
      <p>🕒 Time: {{ timer }} seconds</p>
      <p>Clue: {{ currentRound + 1 }} / 5</p>

      <div class="clue-box" v-if="gameMovie">
        <strong>Clue:</strong> {{ gameMovie.clues[currentRound] }}
      </div>

      <input
        v-model="guess"
        placeholder="Guess the movie"
        @keyup.enter="submitGuess"
        :disabled="gameOver"
      />
      <button @click="submitGuess" :disabled="gameOver">Submit</button>

      <p v-if="message && !gameOver" v-html="message" class="animated-message"></p>
    </div>

    <!-- Popup Modal -->
    <div v-if="gameOver" class="modal">
      <div class="modal-content">
        <h2>{{ messageTitle }}</h2>
        <p v-html="message" class="animated-message"></p>
        <p>The correct answer was: <strong>{{ gameMovie.name }}</strong></p>

        <button @click="shareResult">🔗 Share</button>
        <button @click="resetGame">🔄 Play Again</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const started = ref(false)
const currentRound = ref(0)
const guess = ref('')
const message = ref('')
const messageTitle = ref('')
const gameOver = ref(false)
const timer = ref(0)
let timerInterval = null

const movies = ref([])
const gameMovie = ref(null)
const loading = ref(true)

async function loadMovies() {
  try {
    const response = await fetch('/public/movieData.json')
    if (!response.ok) throw new Error('Failed to fetch movies')

    movies.value = await response.json()
  } catch (e) {
    console.error('Failed to load movies:', e)
    message.value = 'Error loading movie data. Please try again later.'
  } finally {
    loading.value = false
  }
}

function pickRandomMovie() {
  if (movies.value.length === 0) return null
  const idx = Math.floor(Math.random() * movies.value.length)
  return movies.value[idx]
}

function startGame() {
  if (movies.value.length === 0) {
    message.value = 'Movie data not loaded yet.'
    return
  }

  gameMovie.value = pickRandomMovie()
  started.value = true
  gameOver.value = false
  currentRound.value = 0
  guess.value = ''
  message.value = ''
  messageTitle.value = ''
  timer.value = 0

  timerInterval = setInterval(() => {
    timer.value++
  }, 1000)
}

function submitGuess() {
  if (gameOver.value) return

  const userGuess = guess.value.trim().toLowerCase()
  const correctAnswer = gameMovie.value.name.toLowerCase()

  if (userGuess === correctAnswer) {
    messageTitle.value = '🎉 You WON!'
    message.value = `✅ Correct... You WON !!!<br>⏱️ You took ${timer.value} seconds`
    gameOver.value = true
    clearInterval(timerInterval)
    return
  } else {
    currentRound.value++
    if (currentRound.value >= 5) {
      messageTitle.value = '💔 Game Over'
      message.value = '❌ You used all 5 clues!'
      gameOver.value = true
      clearInterval(timerInterval)
      return
    } else {
      message.value = '❌ Wrong guess. Try the next clue! Next clue in 5 seconds...'
    }
  }

  guess.value = ''
}

function resetGame() {
  started.value = false
  gameOver.value = false
  clearInterval(timerInterval)
  message.value = ''
  messageTitle.value = ''
}


function shareResult() {
  const resultText = `🎬 Movie Blasters - I ${messageTitle.value.includes('WON') ? 'won' : 'lost'}!\nMovie: ${gameMovie.value.name}\nTime: ${timer.value} seconds`

  if (navigator.share) {
    navigator
      .share({
        title: 'Movie Blasters Result',
        text: resultText
      })
      .catch(err => console.error('Share failed:', err))
  } else {
    navigator.clipboard.writeText(resultText)
    alert('Result copied to clipboard!')
  }
}

onMounted(() => {
  loadMovies()
})
</script>

<style scoped>
.container {
  z-index: 1;
  background-color: rgb(0, 0, 0);
  padding: 40px 30px;
  border-radius: 7%;
  color: white;
  text-align: center;
  box-shadow: 0 0 10px 4px #07a907;
}

.clue-box {
  border: 2px solid #228b22;
  padding: 15px;
  margin: 20px 0;
  font-size: 1.2em;
  border-radius: 8px;
  background-color: rgba(34, 139, 34, 0.2);
}

input {
  padding: 12px;
  font-size: 18px;
  width: 70%;
  margin-top: 15px;
  border-radius: 6px;
  border: none;
  outline: none;
}

button {
  padding: 12px 24px;
  margin: 10px 5px;
  cursor: pointer;
  border-radius: 8px;
  border: none;
  background-color: #228b22;
  color: white;
  font-weight: bold;
  font-size: 16px;
  transition: background-color 0.3s ease;
}

button:disabled {
  background-color: gray;
  cursor: not-allowed;
}

button:hover:not(:disabled) {
  background-color: #1e6f1e;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 999;
}

.modal-content {
  background: #ffffff;
  color: #000;
  padding: 30px;
  border-radius: 12px;
  text-align: center;
  max-width: 400px;
  width: 90%;
  box-shadow: 0 0 15px #00ff00;
  animation: fadeInPop 0.5s ease;
}

@keyframes popFade {
  0% {
    opacity: 0;
    transform: scale(0.8);
  }
  60% {
    opacity: 1;
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}

@keyframes fadeInPop {
  0% {
    opacity: 0;
    transform: scale(0.8);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}

.animated-message {
  animation: popFade 0.6s ease-out;
}
</style>
