<template>
  <div class="relax-page container-fluid">
    <div class="row g-4">
      
      <!-- ЛЕВАЯ КОЛОНКА: ДЫХАНИЕ И ТАЙМЕР -->
      <div class="col-lg-4">
        <!-- ТРЕНАЖЕР ДЫХАНИЯ -->
        <div class="relax-card p-4 rounded-5 bg-white shadow-sm mb-4">
          <h3 class="h5 fw-bold mb-3 primary-text">🌱 Ритм дыхания</h3>
          <div class="breathing-space">
             <div :class="['breathe-circle', breatheStatus]"></div>
             <p class="breathe-text fw-bold">{{ breatheText }}</p>
          </div>
          <button @click="toggleBreathing" class="btn btn-lavender w-100 rounded-4">
            {{ isBreathing ? 'Остановить' : 'Начать практику' }}
          </button>
        </div>

        <!-- УМНЫЙ POMODORO (Фоновый режим) -->
        <div class="relax-card p-4 rounded-5 bg-white shadow-sm">
          <h3 class="h5 fw-bold mb-1">⏱️ Таймер Помодоро</h3>
          <p class="very-small text-muted mb-3">{{ isBreak ? 'Перерыв' : 'Фокус на задаче' }}</p>
          
          <div class="timer-display display-4 fw-bold mb-3">
            {{ formatTime(timeLeft) }}
          </div>

          <div class="d-flex gap-2">
            <button @click="toggleTimer" class="btn btn-lavender flex-grow-1 rounded-4">
              {{ isTimerRunning ? 'Пауза' : 'Старт' }}
            </button>
            <button @click="resetTimer" class="btn btn-light rounded-4">↺</button>
          </div>
        </div>
      </div>

      <!-- ПРАВАЯ КОЛОНКА: МАТРИЦА ЭЙЗЕНХАУЭРА (Переехала сюда) -->
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

// --- ЛОГИКА ФОНОВОГО ТАЙМЕРА (Senior level) ---
const timeLeft = ref(25 * 60); const isTimerRunning = ref(false); const isBreak = ref(false)
let timerInterval = null

const toggleTimer = () => {
  isTimerRunning.value = !isTimerRunning.value
  if (isTimerRunning.value) {
    // Сохраняем "Время завершения" (Текущее время + оставшиеся секунды)
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
    
    if (remaining <= 0) {
      switchMode()
    } else {
      timeLeft.value = remaining
    }
  }, 1000)
}

const stopInterval = () => { clearInterval(timerInterval) }

const switchMode = () => {
  stopInterval()
  isBreak.value = !isBreak.value
  timeLeft.value = isBreak.value ? 5 * 60 : 25 * 60
  isTimerRunning.value = false
  localStorage.setItem('pomodoro_is_running', 'false')
}

const resetTimer = () => {
  stopInterval(); isTimerRunning.value = false; isBreak.value = false; timeLeft.value = 25 * 60
  localStorage.removeItem('pomodoro_end_time'); localStorage.setItem('pomodoro_is_running', 'false')
}

const formatTime = (s) => `${Math.floor(s / 60)}:${(s % 60).toString().padStart(2, '0')}`

onMounted(() => {
  const running = localStorage.getItem('pomodoro_is_running') === 'true'
  if (running) {
    const endTime = parseInt(localStorage.getItem('pomodoro_end_time'))
    const remaining = Math.round((endTime - Date.now()) / 1000)
    if (remaining > 0) {
      timeLeft.value = remaining; isTimerRunning.value = true; startInterval()
    } else { switchMode() }
  } else {
    const savedLeft = localStorage.getItem('pomodoro_time_left')
    if (savedLeft) timeLeft.value = parseInt(savedLeft)
  }
})
onUnmounted(() => stopInterval())
</script>

<style scoped>
.breathing-space { height: 150px; display: flex; align-items: center; justify-content: center; position: relative; }
.breathe-circle { width: 40px; height: 40px; background: var(--lavender-light); border-radius: 50%; transition: 4s ease-in-out; }
.breathe-circle.inhale { transform: scale(3); background: var(--secondary); }
.breathe-text { position: absolute; font-size: 0.9rem; color: #555; }
.timer-display { font-variant-numeric: tabular-nums; letter-spacing: -2px; }
.btn-lavender { background: var(--primary); color: white; border: none; transition: 0.3s; }
.btn-lavender:hover { background: #9a84be; }
.very-small { font-size: 0.75rem; }
</style>