<template>
  <div class="meditation-page">
    
    <!-- КРАТКАЯ ИНФО-КАРТОЧКА СВЕРХУ -->
    <div class="intro-card">
      <div class="intro-text">
        <h3>Зачем нужна медитация?</h3>
        <p>Для студента это способ «перезагрузить» мозг после лекций, снизить уровень стресса и улучшить память. Достаточно 5-10 минут в день.</p>
      </div>
      <div class="intro-badge">🍃</div>
    </div>

    <div class="meditation-layout">
      
      <!-- ЛЕВАЯ КОЛОНКА: Плеер и Техники -->
      <div class="meditation-main">
        
        <!-- МУЗЫКАЛЬНЫЙ ПЛЕЕР -->
        <div class="meditation-card player-card">
          <div class="player-header">
            <div class="track-info">
              <span class="status-dot" :class="{ 'is-playing': isPlaying }"></span>
              <h3>{{ tracks[currentTrackIndex].name }}</h3>
              <p>{{ isPlaying ? 'Атмосфера запущена...' : 'Выберите звук для практики' }}</p>
            </div>
            <div class="volume-box">
              <span class="vol-icon">🔉</span>
              <input type="range" min="0" max="1" step="0.1" v-model="volume" @input="updateVolume" class="volume-slider">
            </div>
          </div>
          
          <div class="track-selector">
            <button 
              v-for="(track, index) in tracks" 
              :key="index"
              @click="selectTrack(index)"
              :class="['track-btn', { active: currentTrackIndex === index }]"
            >
              {{ track.icon }} {{ track.name }}
            </button>
          </div>

          <button @click="toggleMusic" class="main-play-btn">
            {{ isPlaying ? '⏸ Пауза' : '▶ Слушать атмосферу' }}
          </button>
        </div>

        <!-- ТЕХНИКИ МЕДИТАЦИИ С ОПИСАНИЕМ -->
        <div class="techniques-grid">
          <div v-for="(tech, i) in techniques" :key="i" class="tech-card">
            <span class="tech-emoji">{{ tech.emoji }}</span>
            <h4>{{ tech.title }}</h4>
            <p class="tech-desc">{{ tech.desc }}</p>
            <div class="tech-info-mini">
              <strong>Эффект:</strong> {{ tech.effect }}
            </div>
            <button @click="recordSession(tech.title)" class="record-btn">Завершить практику</button>
          </div>
        </div>
      </div>

      <!-- ПРАВАЯ КОЛОНКА: История -->
      <div class="meditation-sidebar">
        <div class="meditation-card history-card">
          <h3>История сессий</h3>
          <div v-if="history.length" class="history-list">
            <div v-for="(item, i) in history" :key="i" class="history-item">
              <div class="history-dot"></div>
              <div class="history-info">
                <strong>{{ item.name }}</strong>
                <span>{{ item.date }}</span>
              </div>
            </div>
          </div>
          <div v-else class="empty-history">
            <p>Вы еще не проводили медитации. 🌿</p>
          </div>
          <button v-if="history.length" @click="clearHistory" class="clear-btn">Очистить историю</button>
        </div>
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

// ПУТИ К ТВОИМ ЛОКАЛЬНЫМ ФАЙЛАМ В ПАПКЕ PUBLIC
const tracks = [
  { name: 'Шум дождя', url: '/rain.mp3', icon: '🌧️' },
  { name: 'Звуки леса', url: '/forest.mp3', icon: '🌲' },
  { name: 'Дзен-покой', url: '/zen.mp3', icon: '✨' }
]

const currentTrackIndex = ref(0)
const isPlaying = ref(false)
const volume = ref(0.5)
let audio = null

const setupAudio = () => {
  if (audio) {
    audio.pause()
    audio = null
  }
  audio = new Audio(tracks[currentTrackIndex.value].url)
  audio.loop = true
  audio.volume = volume.value
}

const toggleMusic = () => {
  if (!audio) setupAudio()
  
  if (isPlaying.value) {
    audio.pause()
    isPlaying.value = false
  } else {
    audio.play()
    isPlaying.value = true
  }
}

const selectTrack = (index) => {
  const wasPlaying = isPlaying.value
  currentTrackIndex.value = index
  setupAudio()
  if (wasPlaying) {
    audio.play()
  }
}

const updateVolume = () => {
  if (audio) audio.volume = volume.value
}

const techniques = [
  { 
    title: 'Сканирование тела', 
    desc: 'Мысленное наблюдение за каждой частью тела для снятия физических зажимов.', 
    effect: 'Глубокое расслабление',
    emoji: '🧘' 
  },
  { 
    title: 'Дыхание 4-7-8', 
    desc: 'Специальный ритм дыхания, который мгновенно успокаивает нервную систему.', 
    effect: 'Борьба с тревогой',
    emoji: '🌬️' 
  },
  { 
    title: 'Осознанность', 
    desc: 'Практика наблюдения за моментом здесь и сейчас без оценки своих мыслей.', 
    effect: 'Ясность ума',
    emoji: '🍃' 
  }
]

const history = ref([])

const recordSession = (name) => {
  const newSession = {
    name: name,
    date: new Date().toLocaleString('ru-RU', { day: 'numeric', month: 'short', hour: '2-digit', minute: '2-digit' })
  }
  history.value.unshift(newSession)
  localStorage.setItem('meditationHistory', JSON.stringify(history.value))
}

const clearHistory = () => {
  history.value = []
  localStorage.removeItem('meditationHistory')
}

onMounted(() => {
  const saved = localStorage.getItem('meditationHistory')
  if (saved) history.value = JSON.parse(saved)
})

onUnmounted(() => {
  if (audio) audio.pause()
})
</script>

<style scoped>
.intro-card {
  background: white; padding: 30px; border-radius: 35px;
  display: flex; justify-content: space-between; align-items: center;
  margin-bottom: 30px; box-shadow: 0 5px 20px rgba(0,0,0,0.02);
  border: 1px solid #f9f9f9; text-align: left;
}
.intro-text h3 { color: var(--primary); margin-bottom: 8px; }
.intro-text p { color: #888; font-size: 0.95rem; line-height: 1.5; }
.intro-badge { font-size: 2.5rem; opacity: 0.3; }

.meditation-layout { display: grid; grid-template-columns: 1fr 350px; gap: 30px; align-items: flex-start; }
.meditation-card { background: white; padding: 30px; border-radius: 35px; box-shadow: 0 10px 30px rgba(0,0,0,0.03); }

/* PLAYER */
.player-card { border: 2px solid var(--lavender-light); text-align: left; margin-bottom: 30px; }
.player-header { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 25px; }
.status-dot { width: 10px; height: 10px; background: #eee; border-radius: 50%; display: inline-block; margin-right: 10px; }
.status-dot.is-playing { background: #baffc9; box-shadow: 0 0 10px #baffc9; animation: pulse 2s infinite; }

.track-info h3 { color: var(--text-dark); margin-bottom: 5px; }
.track-info p { font-size: 0.8rem; color: #aaa; }

.volume-box { display: flex; flex-direction: column; align-items: center; gap: 5px; }
.volume-slider { accent-color: var(--primary); width: 80px; }

.track-selector { display: flex; gap: 10px; margin-bottom: 25px; flex-wrap: wrap; }
.track-btn { 
  padding: 10px 18px; border-radius: 15px; border: 1px solid #eee; 
  background: #fdfcfd; cursor: pointer; font-family: inherit; font-size: 0.85rem; transition: 0.3s;
}
.track-btn.active { background: var(--primary); color: white; border-color: var(--primary); }

.main-play-btn {
  width: 100%; padding: 15px; border-radius: 15px; border: none;
  background: var(--lavender-light); color: var(--primary); font-weight: 800; font-size: 1.1rem; cursor: pointer; transition: 0.3s;
}

/* TECHNIQUES */
.techniques-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(260px, 1fr)); gap: 20px; }
.tech-card { background: white; padding: 25px; border-radius: 30px; text-align: center; border: 1px solid #fcfcfc; transition: 0.3s; box-shadow: 0 5px 15px rgba(0,0,0,0.02); }
.tech-card:hover { transform: translateY(-5px); border-color: var(--lavender-light); }
.tech-emoji { font-size: 2.5rem; display: block; margin-bottom: 15px; }
.tech-card h4 { margin-bottom: 10px; color: #444; }
.tech-desc { font-size: 0.85rem; color: #777; line-height: 1.5; margin-bottom: 15px; min-height: 45px; }
.tech-info-mini { font-size: 0.75rem; color: var(--primary); background: var(--lavender-light); padding: 5px 10px; border-radius: 8px; margin-bottom: 15px; display: inline-block; }
.record-btn { width: 100%; background: none; border: 1px solid var(--lavender-light); color: var(--primary); padding: 8px; border-radius: 10px; cursor: pointer; font-weight: 700; transition: 0.3s; }
.record-btn:hover { background: var(--lavender-light); }

/* HISTORY */
.history-card h3 { margin-bottom: 20px; text-align: left; }
.history-list { display: flex; flex-direction: column; gap: 15px; text-align: left; }
.history-item { display: flex; gap: 12px; align-items: center; }
.history-dot { width: 6px; height: 6px; background: var(--primary); border-radius: 50%; }
.history-info strong { font-size: 0.85rem; color: #555; display: block; }
.history-info span { font-size: 0.7rem; color: #bbb; }

.clear-btn { margin-top: 25px; background: none; border: none; color: #ccc; cursor: pointer; text-decoration: underline; font-size: 0.8rem; }

@keyframes pulse { 0% { opacity: 1; } 50% { opacity: 0.4; } 100% { opacity: 1; } }

@media (max-width: 1000px) {
  .meditation-layout { grid-template-columns: 1fr; }
}
</style>