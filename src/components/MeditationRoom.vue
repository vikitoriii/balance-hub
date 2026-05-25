<template>
  <div class="meditation-page">
    
    <div class="intro-card">
      <div class="intro-text">
        <h3>Зачем это нужно именно вам? 🧠</h3>
        <p>Медитация — это «тренажерный зал» для мозга. Для студента это способ снизить активность миндалевидного тела (центра страха и паники) и укрепить префронтальную кору, отвечающую за концентрацию. Регулярная практика помогает не просто успокоиться, а научиться сохранять фокус на экзаменах и во время подготовки.</p>
      </div>
      <div class="intro-badge">🧘‍♀️</div>
    </div>

    <div class="meditation-layout">
      
      <div class="meditation-main">
        <!-- МУЗЫКАЛЬНЫЙ ПЛЕЕР -->
        <div class="meditation-card player-card">
          <div class="player-header">
            <div class="track-info">
              <div class="title-row">
                <span class="status-dot" :class="{ 'is-playing': isPlaying }"></span>
                <h3>{{ tracks[currentTrackIndex].name }}</h3>
              </div>
              <!-- ПОДРОБНОЕ ОПИСАНИЕ ВЫБРАННОГО ТРЕКА -->
              <p class="track-full-desc">{{ tracks[currentTrackIndex].desc }}</p>
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

        <!-- ТЕХНИКИ -->
        <div class="techniques-grid">
          <div v-for="(tech, i) in techniques" :key="i" class="tech-card">
            <span class="tech-emoji">{{ tech.emoji }}</span>
            <h4>{{ tech.title }}</h4>
            <div class="tech-info-mini"><strong>Цель:</strong> {{ tech.purpose }}</div>
            <div class="tech-description-full">
              <p class="tech-desc"><strong>Что это:</strong> {{ tech.desc }}</p>
              <p class="tech-how"><strong>Как делать лучше:</strong> {{ tech.how }}</p>
            </div>
            <button @click="recordSession(tech.title)" class="record-btn">Завершить и сохранить</button>
          </div>
        </div>
      </div>

      <!-- ПРАВАЯ КОЛОНКА -->
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
          <div v-else class="empty-history">🌿 Ваша история пока пуста</div>
          <button v-if="history.length" @click="clearHistory" class="clear-btn">Очистить</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const tracks = ref([
  { 
    name: 'Шум дождя', 
    url: '/rain.mp3', 
    icon: '🌧️',
    desc: 'Белый шум дождя помогает создать «эффект кокона», блокируя внешние раздражители. Идеально для концентрации на учебе.'
  },
  { 
    name: 'Звуки леса', 
    url: '/forest.mp3', 
    icon: '🌲',
    desc: 'Стимулирует альфа-волны мозга, которые отвечают за творчество и спокойное бодрствование. Помогает при апатии.'
  },
  { 
    name: 'Дзен-покой', 
    url: '/zen.mp3', 
    icon: '✨',
    desc: 'Мягкий эмбиент для глубокого погружения в себя. Помогает замедлить пульс и подготовить нервную систему к отдыху.'
  },
  { 
    name: 'Шум моря', 
    url: '/sea.mp3', 
    icon: '🌊',
    desc: 'Ритм волн синхронизируется с дыханием, помогая избавиться от чувства тревоги и «смыть» накопленный за день стресс.'
  }
])

const currentTrackIndex = ref(0)
const isPlaying = ref(false)
const volume = ref(0.5)
let audio = null

const setupAudio = () => {
  if (audio) { audio.pause(); audio = null; }
  audio = new Audio(tracks.value[currentTrackIndex.value].url)
  audio.loop = true
  audio.volume = volume.value
}

const toggleMusic = () => {
  if (!audio) setupAudio()
  if (isPlaying.value) {
    audio.pause(); isPlaying.value = false;
  } else {
    audio.play(); isPlaying.value = true;
  }
}

const selectTrack = (index) => {
  const wasPlaying = isPlaying.value
  currentTrackIndex.value = index
  setupAudio()
  if (wasPlaying) audio.play()
}

const updateVolume = () => { if (audio) audio.volume = volume.value }

const techniques = [
  { 
    title: 'Body Scan', 
    purpose: 'Снятие физических зажимов',
    desc: 'Это практика «просмотра» своего тела внутренним взором. Помогает найти, где вы прячете стресс.', 
    how: 'Лягте ровно. Начните с кончиков пальцев ног. Медленно «дышите» через каждую зону, отпуская напряжение.',
    emoji: '🧘' 
  },
  { 
    title: 'Дыхание 4-7-8', 
    purpose: 'Успокоение за 60 секунд',
    desc: 'Методика переключает тело из режима «бей или беги» в режим «отдыхай».', 
    how: 'Вдох носом на 4 счета, задержка на 7, шумный выдох через рот на 8. Это «естественный транквилизатор».',
    emoji: '🌬️' 
  },
  { 
    title: 'Mindfulness', 
    purpose: 'Ясность ума и фокус',
    desc: 'Практика безоценочного наблюдения за своими мыслями.', 
    how: 'Сядьте прямо. Фокусируйтесь на вдохе. Когда придет мысль, просто заметьте её и вернитесь к дыханию.',
    emoji: '🍃' 
  }
]

const history = ref([])
const recordSession = (name) => {
  const newSession = { name, date: new Date().toLocaleString('ru-RU', { day: 'numeric', month: 'short', hour: '2-digit', minute: '2-digit' }) }
  history.value.unshift(newSession); localStorage.setItem('meditationHistory', JSON.stringify(history.value))
}
const clearHistory = () => { history.value = []; localStorage.removeItem('meditationHistory') }
onMounted(() => { const saved = localStorage.getItem('meditationHistory'); if (saved) history.value = JSON.parse(saved) })
onUnmounted(() => { if (audio) audio.pause() })
</script>

<style scoped>
.intro-card { background: white; padding: 30px; border-radius: 35px; display: flex; gap: 20px; margin-bottom: 30px; box-shadow: 0 5px 20px rgba(0,0,0,0.02); text-align: left; }
.intro-text h3 { color: var(--primary); margin-bottom: 10px; font-size: 1.4rem; font-weight: 800; }
.intro-text p { color: #666; font-size: 0.95rem; line-height: 1.6; }
.intro-badge { font-size: 3rem; }

.meditation-layout { display: grid; grid-template-columns: 1fr 320px; gap: 30px; align-items: flex-start; }
.meditation-card { background: white; padding: 30px; border-radius: 35px; box-shadow: 0 10px 30px rgba(0,0,0,0.03); }

.player-card { border: 2px solid var(--lavender-light); text-align: left; margin-bottom: 30px; }
.player-header { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 25px; }
.status-dot { width: 10px; height: 10px; background: #eee; border-radius: 50%; display: inline-block; margin-right: 10px; }
.status-dot.is-playing { background: #baffc9; box-shadow: 0 0 10px #baffc9; animation: pulse 2s infinite; }
.track-info h3 { color: #333; margin-bottom: 5px; }
.track-full-desc { font-size: 0.85rem; color: #888; line-height: 1.4; max-width: 80%; }

.volume-box { display: flex; flex-direction: column; align-items: center; gap: 5px; }
.volume-slider { accent-color: var(--primary); width: 80px; cursor: pointer; }

.track-selector { display: flex; gap: 8px; margin-bottom: 25px; flex-wrap: wrap; }
.track-btn { padding: 10px 15px; border-radius: 12px; border: 1px solid #eee; background: #fdfcfd; cursor: pointer; font-size: 0.8rem; font-weight: 700; transition: 0.3s; }
.track-btn.active { background: var(--primary); color: white; border-color: var(--primary); }

.main-play-btn { width: 100%; padding: 15px; border-radius: 15px; border: none; background: var(--lavender-light); color: var(--primary); font-weight: 800; cursor: pointer; transition: 0.3s; }

.techniques-grid { display: grid; grid-template-columns: 1fr; gap: 20px; }
.tech-card { background: white; padding: 30px; border-radius: 30px; text-align: left; border: 1px solid #fcfcfc; box-shadow: 0 5px 15px rgba(0,0,0,0.02); }
.tech-emoji { font-size: 2.5rem; display: block; margin-bottom: 15px; }
.tech-card h4 { margin-bottom: 10px; font-size: 1.3rem; color: #333; }
.tech-info-mini { font-size: 0.8rem; color: var(--primary); background: var(--lavender-light); padding: 5px 12px; border-radius: 10px; margin-bottom: 15px; display: inline-block; font-weight: 800; }
.tech-description-full { display: flex; flex-direction: column; gap: 12px; margin-bottom: 20px; }
.tech-desc, .tech-how { font-size: 0.95rem; color: #555; line-height: 1.5; }
.record-btn { width: 100%; background: none; border: 1px solid var(--lavender-light); color: var(--primary); padding: 10px; border-radius: 12px; cursor: pointer; font-weight: 700; }

.history-card h3 { margin-bottom: 20px; text-align: left; font-size: 1.1rem; }
.history-list { display: flex; flex-direction: column; gap: 12px; text-align: left; }
.history-item { display: flex; gap: 10px; align-items: center; padding-bottom: 10px; border-bottom: 1px solid #f9f9f9; }
.history-dot { width: 6px; height: 6px; background: var(--primary); border-radius: 50%; }
.history-info strong { font-size: 0.85rem; color: #555; display: block; }
.history-info span { font-size: 0.7rem; color: #bbb; }
.clear-btn { margin-top: 20px; background: none; border: none; color: #ccc; cursor: pointer; text-decoration: underline; font-size: 0.75rem; }

@keyframes pulse { 0% { opacity: 1; } 50% { opacity: 0.4; } 100% { opacity: 1; } }
@media (max-width: 1000px) { .meditation-layout { grid-template-columns: 1fr; } }
</style>