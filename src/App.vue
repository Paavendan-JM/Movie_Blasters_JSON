<template>
  <div class="container">
    <h1>🎬 Movie Blasters</h1>

    <!-- Instructions + Language selection before starting -->
    <div v-if="!started">
      <div class="instructions">
        <h2>📝 Instructions</h2>
        <ul>
          <li>Guess the movie using up to 5 clues.</li>
          <li>Each wrong guess advances the clue.</li>
          <li>Penalty of 5 seconds will be added for each wrong guess.</li>
          <li>You lose if all 5 clues are used.</li>
          <li>Select a language to begin.</li>
        </ul>
      </div>

      <div class="language-buttons">
        <button
          v-for="lang in availableLanguages"
          :key="lang"
          @click="selectLanguage(lang)"
          :class="{ active: selectedLanguage === lang }"
        >
          {{ lang }}
        </button>
      </div>

      <button @click="startGame" :disabled="!selectedLanguage || loading">
        {{ loading ? 'Loading movies...' : 'Start Game' }}
      </button>
    </div>

    <!-- Game screen -->
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

    <!-- Game Over Modal -->
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

const loading = ref(true)
const gameMovie = ref(null)
const movies = ref([])
const selectedLanguage = ref('')
const availableLanguages = ref(['English', 'Hindi', 'Tamil', 'Malayalam'])
const languageMap = ref({})

// Load JSON on mount
onMounted(async () => {
  try {
    const [movieRes, langRes] = await Promise.all([
      fetch('/movieData.json'),
      fetch('/languageMap.json')
    ])
    if (!movieRes.ok || !langRes.ok) throw new Error('Failed to fetch data')
    movies.value = await movieRes.json()
    languageMap.value = await langRes.json()
  } catch (err) {
    console.error('Error loading data:', err)
    message.value = 'Failed to load movie data. Please try again.'
  } finally {
    loading.value = false
  }
})

function selectLanguage(lang) {
  selectedLanguage.value = lang
}

function startGame() {
  const validIds = languageMap.value[selectedLanguage.value] || []
  const filtered = movies.value.filter(m => validIds.includes(m.id))

  if (filtered.length === 0) {
    message.value = 'No movies available in this language.'
    return
  }

  gameMovie.value = filtered[Math.floor(Math.random() * filtered.length)]
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
  timer.value += 5  

  if (currentRound.value >= 5) {
    messageTitle.value = '💔 Game Over'
    message.value = '❌ You used all 5 clues!'
    gameOver.value = true
    clearInterval(timerInterval)
    return
  } else {
    message.value = '❌ Wrong guess. +5 seconds penalty. Try the next clue!'
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
  const result = `🎬 Movie Blasters - I ${messageTitle.value.includes('WON') ? 'won' : 'lost'}!\nMovie: ${gameMovie.value.name}\nTime: ${timer.value} seconds`
  if (navigator.share) {
    navigator.share({ title: 'Movie Blasters', text: result })
  } else {
    navigator.clipboard.writeText(result)
    alert('Result copied to clipboard!')
  }
}
</script>

<style scoped>
.container {
  z-index: 1;
  background-color: black;
  padding: 40px 30px;
  border-radius: 7%;
  color: white;
  text-align: center;
  box-shadow: 0 0 10px 4px #07a907;
}

.instructions {
  background-color: rgba(255, 255, 255, 0.05);
  padding: 20px;
  border-radius: 10px;
  margin-bottom: 20px;
  text-align: left;
}

.instructions ul {
  list-style: disc;
  padding-left: 20px;
}

.language-buttons button {
  margin: 5px;
}

.language-buttons .active {
  background-color: #1e6f1e;
  border: 2px solid #fff;
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
  animation: fadeInPop 0.6s ease-out;
}
</style>
