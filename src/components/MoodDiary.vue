<template>
  <div class="mood-diary">
    <div class="mood-grid">
      
      <!-- ЛЕВАЯ КОЛОНКА: Ввод данных -->
      <div class="mood-sidebar">
        <div class="diary-card add-record">
          <h3>Как прошел день?</h3>
          <p class="subtitle">Отметьте настроение и запишите мысли</p>
          
          <div class="emoji-selector">
            <button 
              v-for="m in moodTypes" 
              :key="m.val"
              @click="selectedMood = m"
              :class="['mood-btn', { active: selectedMood.val === m.val }]"
              :style="{ background: selectedMood.val === m.val ? m.color : '' }"
            >
              <span class="m-emoji">{{ m.emoji }}</span>
              <span class="m-label">{{ m.label }}</span>
            </button>
          </div>

          <div class="input-group">
            <label>Что вы чувствуете?</label>
            <textarea v-model="note" placeholder="Опишите ваши эмоции или события..."></textarea>
          </div>

          <button @click="saveEntry" class="save-btn" :disabled="!selectedMood.val">
            Зафиксировать состояние
          </button>
        </div>

        <!-- КРАТКАЯ СТАТИСТИКА -->
        <div class="diary-card stats-mini">
          <h4>Ваша неделя</h4>
          <div class="stats-row">
            <div class="stat-item">
              <span>Записей</span>
              <strong>{{ entries.length }}</strong>
            </div>
            <div class="stat-item">
              <span>Среднее</span>
              <strong>{{ averageMoodEmoji }}</strong>
            </div>
          </div>
        </div>
      </div>

      <!-- ПРАВАЯ КОЛОНКА: Аналитика и История -->
      <div class="mood-main">
        
        <!-- ГРАФИК (Chart.js) -->
        <div class="diary-card chart-container">
          <div class="chart-header">
            <h3>Аналитика настроения</h3>
            <p>Динамика за последние 7 записей</p>
          </div>
          <div class="canvas-wrapper">
            <canvas id="moodChart"></canvas>
          </div>
        </div>

        <!-- СПИСОК ЗАПИСЕЙ -->
        <div class="history-section">
          <h3>История записей</h3>
          <div v-if="entries.length" class="entries-list">
            <div v-for="entry in entries" :key="entry.id" class="entry-card">
              <div class="entry-mood" :style="{ background: entry.color }">{{ entry.emoji }}</div>
              <div class="entry-info">
                <div class="entry-header">
                  <span class="entry-date">{{ entry.date }}</span>
                  <span class="entry-label">{{ entry.label }}</span>
                </div>
                <p class="entry-text">{{ entry.text || 'Без описания' }}</p>
              </div>
              <button @click="deleteEntry(entry.id)" class="del-entry">×</button>
            </div>
          </div>
          <div v-else class="empty-state">
            <p>Начните вести дневник, чтобы увидеть статистику 🌿</p>
          </div>
        </div>

      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, watch, nextTick } from 'vue'
import Chart from 'chart.js/auto' // Импорт библиотеки графиков

// Типы настроения
const moodTypes = [
  { val: 1, emoji: '😔', label: 'Тяжело', color: '#ffb3ba' },
  { val: 2, emoji: '😕', label: 'Так себе', color: '#ffdfba' },
  { val: 3, emoji: '😐', label: 'Нормально', color: '#ffffba' },
  { val: 4, emoji: '🙂', label: 'Хорошо', color: '#baffc9' },
  { val: 5, emoji: '🤩', label: 'Отлично', color: '#bae1ff' }
]

const selectedMood = ref({ val: null })
const note = ref('')
const entries = ref([])
let moodChart = null

// Сохранение записи
const saveEntry = () => {
  const newEntry = {
    id: Date.now(),
    val: selectedMood.value.val,
    emoji: selectedMood.value.emoji,
    label: selectedMood.value.label,
    color: selectedMood.value.color,
    text: note.value,
    date: new Date().toLocaleDateString('ru-RU', { day: 'numeric', month: 'short' })
  }
  
  entries.value.unshift(newEntry)
  localStorage.setItem('moodData', JSON.stringify(entries.value))
  
  // Сброс формы
  note.value = ''
  selectedMood.value = { val: null }
  
  updateChart()
}

// Удаление
const deleteEntry = (id) => {
  entries.value = entries.value.filter(e => e.id !== id)
  localStorage.setItem('moodData', JSON.stringify(entries.value))
  updateChart()
}

// СРЕДНЕЕ НАСТРОЕНИЕ (для статистики)
const averageMoodEmoji = computed(() => {
  if (!entries.value.length) return '—'
  const avg = Math.round(entries.value.reduce((acc, e) => acc + e.val, 0) / entries.value.length)
  return moodTypes.find(m => m.val === avg)?.emoji || '😐'
})

// ЛОГИКА ГРАФИКА (Требование №5: Обработка данных)
const initChart = () => {
  const ctx = document.getElementById('moodChart')
  if (!ctx) return

  moodChart = new Chart(ctx, {
    type: 'line',
    data: {
      labels: [],
      datasets: [{
        label: 'Уровень настроения',
        data: [],
        borderColor: '#b39cd0',
        backgroundColor: 'rgba(179, 156, 208, 0.2)',
        tension: 0.4, // Делает линию плавной
        fill: true,
        pointRadius: 6,
        pointBackgroundColor: '#fff',
        pointBorderWidth: 3
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      scales: {
        y: { min: 1, max: 5, ticks: { stepSize: 1, callback: (v) => moodTypes.find(m => m.val === v)?.emoji } },
        x: { grid: { display: false } }
      },
      plugins: { legend: { display: false } }
    }
  })
  updateChart()
}

const updateChart = () => {
  if (!moodChart) return
  const lastEntries = [...entries.value].reverse().slice(-7)
  moodChart.data.labels = lastEntries.map(e => e.date)
  moodChart.data.datasets[0].data = lastEntries.map(e => e.val)
  moodChart.update()
}

onMounted(() => {
  const saved = localStorage.getItem('moodData')
  if (saved) entries.value = JSON.parse(saved)
  nextTick(() => initChart())
})
</script>

<style scoped>
.mood-grid { display: flex; gap: 40px; align-items: flex-start; justify-content: center; }
.mood-sidebar { flex: 0 0 350px; display: flex; flex-direction: column; gap: 20px; }
.mood-main { flex: 1; max-width: 800px; }

.diary-card { background: white; padding: 30px; border-radius: 40px; box-shadow: 0 10px 30px rgba(0,0,0,0.03); border: 1px solid #f9f9f9; }

/* Emoji Selector */
.emoji-selector { display: grid; grid-template-columns: repeat(5, 1fr); gap: 10px; margin-bottom: 25px; }
.mood-btn { 
  display: flex; flex-direction: column; align-items: center; padding: 10px; 
  border-radius: 15px; border: 1px solid #eee; background: white; cursor: pointer; transition: 0.3s;
}
.m-emoji { font-size: 1.5rem; margin-bottom: 5px; }
.m-label { font-size: 0.6rem; text-transform: uppercase; font-weight: 800; color: #999; }
.mood-btn.active { border-color: transparent; }
.mood-btn.active .m-label { color: #333; }

.input-group { text-align: left; margin-bottom: 20px; }
.input-group label { display: block; font-weight: 800; font-size: 0.9rem; margin-bottom: 10px; color: #555; }
.input-group textarea { width: 100%; height: 100px; padding: 15px; border-radius: 20px; border: 1px solid #eee; font-family: inherit; outline: none; background: #fdfcfd; }

.save-btn { width: 100%; padding: 15px; border-radius: 15px; border: none; background: var(--primary); color: white; font-weight: 800; cursor: pointer; }
.save-btn:disabled { background: #eee; cursor: not-allowed; }

/* Stats */
.stats-row { display: flex; justify-content: space-around; margin-top: 15px; }
.stat-item { display: flex; flex-direction: column; }
.stat-item span { font-size: 0.75rem; color: #aaa; text-transform: uppercase; }
.stat-item strong { font-size: 1.5rem; color: var(--primary); }

/* Chart */
.chart-container { margin-bottom: 40px; text-align: left; }
.canvas-wrapper { height: 250px; margin-top: 20px; }

/* History */
.history-section h3 { text-align: left; margin-bottom: 20px; }
.entry-card { 
  background: white; padding: 20px; border-radius: 25px; display: flex; align-items: center; gap: 20px;
  margin-bottom: 15px; box-shadow: 0 5px 15px rgba(0,0,0,0.02); position: relative; text-align: left;
}
.entry-mood { width: 50px; height: 50px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 1.5rem; flex-shrink: 0; }
.entry-info { flex: 1; }
.entry-header { display: flex; justify-content: space-between; margin-bottom: 5px; }
.entry-date { font-size: 0.8rem; color: #bbb; font-weight: 700; }
.entry-label { font-size: 0.8rem; font-weight: 800; text-transform: uppercase; color: var(--primary); }
.entry-text { font-size: 0.95rem; color: #666; line-height: 1.4; }
.del-entry { background: none; border: none; color: #ddd; font-size: 1.5rem; cursor: pointer; }

@media (max-width: 1000px) { .mood-grid { flex-direction: column; align-items: center; } .mood-sidebar, .mood-main { width: 100%; } }
</style>