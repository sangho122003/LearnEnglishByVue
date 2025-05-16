<template>
  <div class="app-container">
    <header>
      <h1>Học từ vựng - Campus</h1>
      <div class="campus-select">
        <label>Chọn Campus: </label>
        <select v-model="selectedCampus" @change="resetTest">
          <option v-for="c in campuses" :key="c" :value="c">Campus {{ c }}</option>
        </select>
      </div>
    </header>

    <main v-if="currentWord">
      <div class="skill-display">
        Skill: <span class="skill">{{ currentWord.Skill }}</span>
      </div>

      <div class="word-box">
        <label class="word-label">
          {{ direction ? currentWord.Word : 'Nghĩa: ' + currentWord.Meaning }}
          <!-- Nút nghe từ chỉ hiện khi đang Anh->Việt -->
          <button v-if="direction" @click="speakWord" class="btn-speak" title="Nghe từ này">🔊</button>
        </label>
        <input
          v-model="userInput"
          @keyup.enter="checkAnswer"
          :placeholder="direction ? 'Nhập nghĩa tiếng Việt' : 'Nhập từ tiếng Anh'"
          autocomplete="off"
        />
      </div>

      <button @click="checkAnswer" class="btn-check">Kiểm tra</button>

      <p class="feedback" :class="{ correct: isCorrect === true, wrong: isCorrect === false }">
        {{ feedback }}
      </p>
    </main>

    <main v-else>
      <div class="success-message">
        <h2>Bài thi thành công! 🎉</h2>
        <button @click="resetTest">Chọn lại Campus</button>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const data = ref([])
const campuses = ref([])
const selectedCampus = ref('1')
const index = ref(0)
const userInput = ref('')
const isCorrect = ref(null)
const feedback = ref('')
const direction = ref(true) // true = Anh->Việt, false = Việt->Anh

const filteredData = computed(() =>
  data.value.filter((item) => item.Campus.trim() === `Campus ${selectedCampus.value}`)
)

const currentWord = computed(() => {
  if (!filteredData.value.length) return null
  if (index.value === null) return null
  return filteredData.value[index.value]
})

async function fetchData() {
  const res = await fetch(
    'https://opensheet.elk.sh/1O6mghwbLmjN7FG746rDfxxwfMjmDEgsbJLHCrXb4HgM/Sheet1'
  )
  const json = await res.json()
  data.value = json
  const camps = [...new Set(json.map((item) => item.Campus.trim()))]
  campuses.value = camps.map((c) => c.replace('Campus ', ''))
}

function checkAnswer() {
  if (!userInput.value.trim()) return
  if (!currentWord.value) return

  let correctAnswer = direction.value
    ? currentWord.value.Meaning.trim().toLowerCase() // Anh->Việt
    : currentWord.value.Word.trim().toLowerCase()    // Việt->Anh

  if (userInput.value.trim().toLowerCase() === correctAnswer) {
    isCorrect.value = true
    feedback.value = 'Chính xác! 🎉'
    userInput.value = ''

    if (index.value < filteredData.value.length - 1) {
      index.value++
    } else {
      // Hết chiều hiện tại
      if (direction.value) {
        // Chuyển sang chiều ngược lại
        direction.value = false
        index.value = 0
        feedback.value = 'Chuyển sang chiều ngược lại!'
      } else {
        // Hết cả 2 chiều
        index.value = null
      }
    }
  } else {
    isCorrect.value = false
    feedback.value = 'Sai rồi, thử lại nhé!'
  }
}

function resetTest() {
  index.value = 0
  userInput.value = ''
  isCorrect.value = null
  feedback.value = ''
  direction.value = true
}

// Hàm đọc to từ tiếng Anh
function speakWord() {
  if (!currentWord.value) return
  const utterance = new SpeechSynthesisUtterance(currentWord.value.Word)
  utterance.lang = 'en-US'  // Giọng tiếng Anh Mỹ
  speechSynthesis.speak(utterance)
}

onMounted(() => {
  fetchData()
})
</script>

<style scoped>
.app-container {
  max-width: 400px;
  margin: 20px auto;
  font-family: 'Comic Sans MS', cursive, sans-serif;
  border: 3px solid #fbc02d; /* vàng */
  border-radius: 12px;
  background: linear-gradient(135deg, #2196f3 0%, #f44336 100%);
  color: white;
  padding: 20px;
  box-shadow: 0 0 10px #fbc02d;
}

header {
  margin-bottom: 20px;
  text-align: center;
}

.campus-select {
  margin-top: 10px;
}

select {
  font-size: 1rem;
  border-radius: 6px;
  padding: 4px 8px;
  border: none;
  outline: none;
  cursor: pointer;
}

.skill-display {
  font-size: 1.2rem;
  margin-bottom: 12px;
  font-weight: bold;
  background-color: #fbc02d;
  color: #2196f3;
  padding: 6px 10px;
  border-radius: 8px;
  width: fit-content;
}

.word-box {
  margin-bottom: 12px;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.word-label {
  font-size: 2.4rem;
  font-weight: bold;
  color: #f44336;
  margin-bottom: 8px;
  display: flex;
  align-items: center;
}

input {
  font-size: 1.2rem;
  padding: 8px;
  border-radius: 10px;
  border: 2px solid #fbc02d;
  width: 100%;
  box-sizing: border-box;
}

input:focus {
  outline: none;
  border-color: #2196f3;
}

.btn-check {
  width: 100%;
  background-color: #fbc02d;
  border: none;
  color: #2196f3;
  font-weight: bold;
  font-size: 1.1rem;
  padding: 10px;
  border-radius: 10px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.btn-check:hover {
  background-color: #f9a825;
}

.feedback {
  margin-top: 10px;
  font-weight: bold;
  font-size: 1.1rem;
  height: 30px;
}

.correct {
  color: #4caf50;
}

.wrong {
  color: #ffeb3b;
}

.success-message {
  text-align: center;
  background-color: #4caf50;
  padding: 20px;
  border-radius: 15px;
  box-shadow: 0 0 10px #66bb6a;
}

.success-message h2 {
  margin-bottom: 15px;
  font-size: 1.8rem;
  color: white;
}

.success-message button {
  padding: 12px 24px;
  font-weight: bold;
  font-size: 1.2rem;
  border-radius: 12px;
  border: none;
  cursor: pointer;
  background-color: #fbc02d;
  color: #2196f3;
  transition: background-color 0.3s ease;
}

.success-message button:hover {
  background-color: #f9a825;
}

.btn-speak {
  background: transparent;
  border: none;
  cursor: pointer;
  margin-left: 10px;
  font-size: 1.8rem;
  color: white;
  transition: color 0.3s ease;
}

.btn-speak:hover {
  color: #fbc02d;
}
</style>
