<template>
  <div class="relax-page container-fluid">
    <div class="row g-4">
      
      <!-- ЛЕВАЯ КОЛОНКА: ДЫХАНИЕ, ТАЙМЕР И ЦИТАТЫ -->
      <div class="col-lg-4">
        
        <!-- ТРЕНАЖЕР ДЫХАНИЯ -->
        <div class="relax-card p-4 rounded-5 bg-white shadow-sm mb-4">
          <h3 class="h5 fw-bold mb-3 primary-text">🌱 Ритм дыхания</h3>
          <div class="breathing-space">
             <div :class="['breathe-circle', breatheStatus]"></div>
             <p class="breathe-text fw-bold">{{ breatheText }}</p>
          </div>
          <button @click="toggleBreathing" class="btn btn-lavender w-100 rounded-4 fw-bold">
            {{ isBreathing ? 'Остановить' : 'Начать практику' }}
          </button>
        </div>

        <!-- УМНЫЙ POMODORO (С ПОДСКАЗКОЙ) -->
        <div class="relax-card p-4 rounded-5 bg-white shadow-sm mb-4">
          <div class="d-flex align-items-center justify-content-between mb-1">
            <div class="d-flex align-items-center gap-2">
              <h3 class="h5 fw-bold m-0 text-dark">⏱️ Таймер Помодоро</h3>
              <!-- ВСПЛЫВАЮЩАЯ ПОДСКАЗКА -->
              <div class="hint-wrapper">
                <span class="hint-icon">?</span>
                <div class="hint-content">
                  <strong>Метод 25/5:</strong><br>
                  25 минут — полная фокусировка на одной задаче.<br>
                  5 минут — отдых (встать, размяться).<br>
                  Каждые 4 цикла — длинный перерыв (15 мин).
                </div>
              </div>
            </div>
          </div>
          <p class="very-small text-muted mb-3 fw-bold" :style="{ color: isBreak ? 'var(--primary)' : 'var(--accent)' }">
            {{ isBreak ? 'Режим: Перерыв' : 'Режим: Фокус' }}
          </p>
          
          <div class="timer-display display-4 fw-bold mb-3 text-dark">
            {{ formatTime(timeLeft) }}
          </div>

          <div class="d-flex gap-2">
            <button @click="toggleTimer" class="btn btn-lavender flex-grow-1 rounded-4 fw-bold">
              {{ isTimerRunning ? 'Пауза' : 'Старт' }}
            </button>
            <button @click="resetTimer" class="btn btn-light rounded-4 text-muted">↺</button>
          </div>
        </div>

        <!-- БЛОК МОТИВАЦИИ (ВЕРНУЛИ) -->
        <div class="relax-card p-4 rounded-5 bg-white shadow-sm affirmation-card">
          <div class="card-header-mini mb-3">
             <span class="badge bg-lavender text-primary rounded-pill px-3">Мотивация</span>
          </div>
          <div class="affirmation-content mb-3">
            <Transition name="fade" mode="out-in">
              <p :key="currentQuote" class="affirmation-text italic">
                "{{ currentQuote }}"
              </p>
            </Transition>
          </div>
          <button @click="nextQuote" class="btn btn-light w-100 rounded-4 btn-sm fw-bold text-muted">
             Другое послание ✨
          </button>
        </div>
      </div>

      <!-- ПРАВАЯ КОЛОНКА: МАТРИЦА ЭЙЗЕНХАУЭРА -->
      <div class="col-lg-8">
        <PriorityMatrix />
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import PriorityMatrix from './PriorityMatrix.vue'

// --- ЛОГИКА ДЫХАНИЯ ---
const isBreathing = ref(false); const breatheStatus = ref(''); const breatheText = ref('Вдох-выдох')
let breatheInterval = null
const toggleBreathing = () => {
  isBreathing.value = !isBreathing.value
  if (isBreathing.value) {
    const cycle = () => {
      breatheStatus.value = 'inhale'; breatheText.value = 'Вдох...';
      setTimeout(() => { if (isBreathing.value) { breatheStatus.value = 'exhale'; breatheText.value = 'Выдох...'; } }, 4000)
    }
    cycle(); breatheInterval = setInterval(cycle, 8000)
  } else { clearInterval(breatheInterval); breatheStatus.value = ''; breatheText.value = 'Вдох-выдох' }
}

// --- ЛОГИКА ФОНОВОГО ТАЙМЕРА ---
const timeLeft = ref(25 * 60); const isTimerRunning = ref(false); const isBreak = ref(false)
let timerInterval = null

const toggleTimer = () => {
  isTimerRunning.value = !isTimerRunning.value
  if (isTimerRunning.value) {
    const endTime = Date.now() + (timeLeft.value * 1000)
    localStorage.setItem('pomodoro_end_time', endTime)
    localStorage.setItem('pomodoro_is_running', 'true')
    startInterval()
  } else {
    stopInterval()
    localStorage.setItem('pomodoro_is_running', 'false')
    localStorage.setItem('pomodoro_time_left', timeLeft.value)
  }
}

const startInterval = () => {
  timerInterval = setInterval(() => {
    const endTime = parseInt(localStorage.getItem('pomodoro_end_time'))
    const remaining = Math.round((endTime - Date.now()) / 1000)
    if (remaining <= 0) switchMode()
    else timeLeft.value = remaining
  }, 1000)
}

const stopInterval = () => clearInterval(timerInterval)

const switchMode = () => {
  stopInterval(); isBreak.value = !isBreak.value
  timeLeft.value = isBreak.value ? 5 * 60 : 25 * 60
  isTimerRunning.value = false; localStorage.setItem('pomodoro_is_running', 'false')
}

const resetTimer = () => {
  stopInterval(); isTimerRunning.value = false; isBreak.value = false; timeLeft.value = 25 * 60
  localStorage.removeItem('pomodoro_end_time'); localStorage.setItem('pomodoro_is_running', 'false')
}

const formatTime = (s) => `${Math.floor(s / 60)}:${(s % 60).toString().padStart(2, '0')}`

// --- ЛОГИКА ЦИТАТ ---
const quotes = [
  "Твой успех начинается с тишины внутри тебя.",
  "Делай то, что можешь, с тем, что имеешь, там, где ты есть.",
  "Твое спокойствие — твоя суперсила.",
  "Продуктивность — это не занятость, а умение фокусироваться.",
  "Разреши себе отдыхать, это топливо для твоих побед.",
  "Маленький шаг сегодня — большой результат завтра."
]
const currentQuote = ref(quotes[0])
const nextQuote = () => {
  const currentIndex = quotes.indexOf(currentQuote.value)
  currentQuote.value = quotes[(currentIndex + 1) % quotes.length]
}

onMounted(() => {
  const running = localStorage.getItem('pomodoro_is_running') === 'true'
  if (running) {
    const endTime = parseInt(localStorage.getItem('pomodoro_end_time'))
    const remaining = Math.round((endTime - Date.now()) / 1000)
    if (remaining > 0) { timeLeft.value = remaining; isTimerRunning.value = true; startInterval() }
    else switchMode()
  } else {
    const savedLeft = localStorage.getItem('pomodoro_time_left')
    if (savedLeft) timeLeft.value = parseInt(savedLeft)
  }
})
onUnmounted(() => stopInterval())
</script>

<style scoped>
.breathing-space { height: 140px; display: flex; align-items: center; justify-content: center; position: relative; }
.breathe-circle { width: 40px; height: 40px; background: var(--lavender-light); border-radius: 50%; transition: 4s ease-in-out; box-shadow: 0 0 30px var(--lavender-light); }
.breathe-circle.inhale { transform: scale(2.8); background: var(--secondary); }
.breathe-text { position: absolute; font-size: 0.85rem; color: #555; }

.timer-display { font-variant-numeric: tabular-nums; letter-spacing: -1px; }

/* ПОДСКАЗКИ */
.hint-wrapper { position: relative; }
.hint-icon { 
  width: 16px; height: 16px; background: #eee; border-radius: 50%; 
  display: flex; align-items: center; justify-content: center; 
  font-size: 10px; cursor: help; color: #999; font-weight: 800; 
}
.hint-content { 
  position: absolute; z-index: 100; bottom: 25px; left: 50%; transform: translateX(-50%); 
  width: 220px; background: #2d2d2d; color: #fff; padding: 12px; border-radius: 12px; 
  font-size: 0.75rem; opacity: 0; transition: 0.3s; pointer-events: none; text-align: left;
}
.hint-wrapper:hover .hint-content { opacity: 1; }

/* АФФИРМАЦИИ */
.affirmation-text { font-size: 1rem; line-height: 1.5; color: #666; min-height: 60px; display: flex; align-items: center; justify-content: center; }
.italic { font-style: italic; }

.btn-lavender { background: var(--primary); color: white; border: none; transition: 0.3s; }
.btn-lavender:hover { background: #9a84be; transform: translateY(-2px); }

.fade-enter-active, .fade-leave-active { transition: opacity 0.4s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
</style>