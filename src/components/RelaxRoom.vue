<template>
  <div class="relax-section">
    <div class="relax-grid">
      
      <!-- БЛОК 1: ТРЕНАЖЕР ДЫХАНИЯ -->
      
       <div class="relax-card breathing-box">
        <h3>🌱 Тренажер дыхания</h3>
        <p class="relax-desc">Следуйте за кругом, чтобы успокоить мысли</p>
        
        <div class="breathing-container">
          <div :class="['breathing-circle', breatheStatus]"></div>
          <div class="breathe-text">{{ breatheText }}</div>
        </div>
        
        <button @click="toggleBreathing" class="action-btn">
          {{ isBreathing ? 'Остановить' : 'Начать практику' }}
        </button>
      </div>

      <!-- БЛОК 2: POMODORO ТАЙМЕР -->
      <div class="relax-card pomodoro-card">
        <div class="card-header">
          <h3>⏱️ Помодоро</h3>
          <div class="hint-wrapper">
            <span class="hint-icon">?</span>
            <div class="hint-content">
              <strong>Метод Помидора:</strong><br>
              1. Фокус: 25 мин<br>
              2. Отдых: 5 мин<br>
              3. Каждые 4 цикла: длинный отдых (15 мин).
            </div>
          </div>
        </div>
        <p class="relax-desc" :style="{ color: isBreak ? 'var(--primary)' : 'var(--accent)' }">
          {{ isBreak ? 'Время перерыва' : 'Время фокусировки' }}
        </p>
        
        <div class="timer-display">
          {{ formatTime(timeLeft) }}
        </div>

        <!-- Индикаторы циклов -->
        <div class="cycles-track">
          <span v-for="n in 4" :key="n" 
                :class="['cycle-dot', { active: n <= cyclesCompleted % 4 + (isBreak ? 0 : 0), current: n === (cyclesCompleted % 4 + 1) && !isBreak }]">
          </span>
        </div>
        
        <div class="timer-controls">
          <button @click="toggleTimer" class="timer-btn-main">
            {{ isTimerRunning ? 'Пауза' : 'Старт' }}
          </button>
          <button @click="resetTimer" class="timer-btn-reset">↺</button>
        </div>
      </div>

      <!-- БЛОК 3: УЮТНЫЕ АФФИРМАЦИИ -->
      <div class="relax-card affirmation-card">
        <div class="card-header">
          <h3>✨ Твое состояние</h3>
        </div>
        <p class="relax-desc">Послание для тебя</p>
        
        <div class="affirmation-content">
          <Transition name="fade" mode="out-in">
            <p :key="currentAffirmation" class="affirmation-text">
              "{{ currentAffirmation }}"
            </p>
          </Transition>
        </div>
        
        <button @click="nextAffirmation" class="next-btn">Хочу еще</button>
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, onUnmounted, computed } from 'vue'

// --- ЛОГИКА ДЫХАНИЯ ---
const isBreathing = ref(false)
const breatheStatus = ref('')
const breatheText = ref('Запуск')
let breatheInterval = null

const toggleBreathing = () => {
  isBreathing.value = !isBreathing.value
  if (isBreathing.value) {
    runBreatheCycle()
    breatheInterval = setInterval(runBreatheCycle, 8000)
  } else {
    clearInterval(breatheInterval)
    breatheStatus.value = ''
    breatheText.value = 'Запуск'
  }
}

const runBreatheCycle = () => {
  breatheStatus.value = 'inhale'
  breatheText.value = 'Вдох...'
  setTimeout(() => {
    if(isBreathing.value) {
      breatheStatus.value = 'exhale'
      breatheText.value = 'Выдох...'
    }
  }, 4000)
}

// --- ЛОГИКА POMODORO ---
const timeLeft = ref(25 * 60)
const isTimerRunning = ref(false)
const isBreak = ref(false)
const cyclesCompleted = ref(0)
let timerInterval = null

const toggleTimer = () => {
  isTimerRunning.value = !isTimerRunning.value
  if (isTimerRunning.value) {
    timerInterval = setInterval(() => {
      if (timeLeft.value > 0) {
        timeLeft.value--
      } else {
        switchMode()
      }
    }, 1000)
  } else {
    clearInterval(timerInterval)
  }
}

const switchMode = () => {
  clearInterval(timerInterval)
  isTimerRunning.value = false
  
  if (!isBreak.value) {
    // Был рабочий цикл
    cyclesCompleted.value++
    isBreak.value = true
    // Длинный перерыв после 4 циклов
    timeLeft.value = (cyclesCompleted.value % 4 === 0) ? 15 * 60 : 5 * 60
  } else {
    // Был перерыв
    isBreak.value = false
    timeLeft.value = 25 * 60
  }
  // Автоматический старт следующего режима (опционально)
  toggleTimer()
}

const resetTimer = () => {
  clearInterval(timerInterval)
  isTimerRunning.value = false
  isBreak.value = false
  timeLeft.value = 25 * 60
}

const formatTime = (seconds) => {
  const m = Math.floor(seconds / 60)
  const s = seconds % 60
  return `${m}:${s < 10 ? '0' : ''}${s}`
}

// --- ЛОГИКА АФФИРМАЦИЙ ---
const affirmations = [
  "Я разрешаю себе отдыхать столько, сколько мне нужно",
  "Моя ценность не зависит от моей продуктивности",
  "Я делаю всё, что в моих силах, и этого достаточно",
  "Я выбираю спокойствие вместо тревоги",
  "С каждым вдохом я становлюсь спокойнее",
  "Мои цели достижимы, а мой путь уникален"
]
const currentAffirmation = ref(affirmations[0])
const nextAffirmation = () => {
  const idx = affirmations.indexOf(currentAffirmation.value)
  currentAffirmation.value = affirmations[(idx + 1) % affirmations.length]
}

onUnmounted(() => {
  clearInterval(breatheInterval)
  clearInterval(timerInterval)
})
</script>

<style scoped>
.relax-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
  gap: 30px;
}

.relax-card {
  background: white;
  padding: 35px;
  border-radius: 40px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.03);
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.card-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 5px;
}

h3 { color: var(--primary); font-size: 1.4rem; font-weight: 800; }

/* ПОДСКАЗКИ */
.hint-wrapper { position: relative; }
.hint-icon {
  width: 18px; height: 18px; background: #f0f0f0; border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  font-size: 10px; cursor: help; color: #aaa; font-weight: 800;
}
.hint-content {
  position: absolute; z-index: 10; bottom: 25px; left: 50%; transform: translateX(-50%);
  width: 220px; background: #333; color: #fff; padding: 12px; border-radius: 12px;
  font-size: 0.8rem; opacity: 0; transition: 0.3s; pointer-events: none; text-align: left;
}
.hint-wrapper:hover .hint-content { opacity: 1; }

.relax-desc { font-size: 0.9rem; color: #999; margin-bottom: 30px; font-weight: 600; }

/* ДЫХАНИЕ */
.breathing-container {
  height: 180px; display: flex; align-items: center; justify-content: center; position: relative; margin-bottom: 20px;
}
.breathing-circle {
  width: 50px; height: 60px; background: var(--lavender-light); border-radius: 50%;
  transition: all 4s ease-in-out; box-shadow: 0 0 40px var(--lavender-light);
}
.breathing-circle.inhale { transform: scale(3); background: var(--secondary); }
.breathing-circle.exhale { transform: scale(1); background: var(--lavender-light); }
.breathe-text { position: absolute; font-weight: 800; color: #555; }

/* ТАЙМЕР */
.timer-display { font-size: 4.5rem; font-weight: 800; color: var(--text-dark); margin-bottom: 10px; font-variant-numeric: tabular-nums; }
.cycles-track { display: flex; gap: 8px; margin-bottom: 25px; }
.cycle-dot { width: 10px; height: 10px; background: #eee; border-radius: 50%; transition: 0.3s; }
.cycle-dot.active { background: var(--primary); transform: scale(1.1); }
.cycle-dot.current { border: 2px solid var(--accent); animation: pulse 2s infinite; }

@keyframes pulse { 0% { opacity: 1; } 50% { opacity: 0.5; } 100% { opacity: 1; } }

.timer-controls { display: flex; gap: 15px; }
.timer-btn-main { background: var(--primary); color: white; border: none; padding: 12px 35px; border-radius: 15px; font-weight: 700; cursor: pointer; transition: 0.3s; }
.timer-btn-reset { background: #f5f5f5; color: #ccc; border: none; width: 45px; border-radius: 15px; font-size: 1.2rem; cursor: pointer; }

/* АФФИРМАЦИИ */
.affirmation-content { height: 120px; display: flex; align-items: center; justify-content: center; margin-bottom: 20px; padding: 0 10px; }
.affirmation-text { font-size: 1.1rem; font-style: italic; color: #555; line-height: 1.5; font-weight: 600; }
.next-btn { background: var(--lavender-light); color: var(--primary); border: none; padding: 10px 25px; border-radius: 12px; font-weight: 700; cursor: pointer; }

.action-btn { background: var(--lavender-light); color: var(--primary); border: none; padding: 12px 30px; border-radius: 15px; font-weight: 700; cursor: pointer; }
.action-btn:hover, .next-btn:hover { background: var(--primary); color: white; }

/* Анимация текста */
.fade-enter-active, .fade-leave-active { transition: opacity 0.5s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
</style>