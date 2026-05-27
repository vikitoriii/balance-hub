<template>
  <div class="meditation-page">
    <div class="intro-mini-card">
      <div class="intro-text">
        <h3>Зачем медитировать? 🍃</h3>
        <p>Для студента это способ «перезагрузить» мозг, снизить стресс и улучшить память. Достаточно 5 минут.</p>
      </div>
    </div>

    <div class="meditation-layout">
      <div class="meditation-main">
        <div class="meditation-card player-card">
          <div class="player-header">
            <div class="track-info">
              <div class="title-row">
                <span class="status-dot" :class="{ 'is-playing': isPlaying }"></span>
                <h4 class="m-0 fw-bold">{{ tracks[currentTrackIndex].name }}</h4>
              </div>
              <p class="track-full-desc">{{ tracks[currentTrackIndex].desc }}</p>
            </div>
            <div class="volume-box">
              <input type="range" min="0" max="1" step="0.1" v-model="volume" @input="updateVolume" class="volume-slider">
            </div>
          </div>
          <div class="track-selector">
            <button v-for="(t, i) in tracks" :key="i" @click="selectTrack(i)" :class="['t-btn', { active: currentTrackIndex === i }]">
              {{ t.icon }} {{ t.name }}
            </button>
          </div>
          <button @click="toggleMusic" class="main-play-btn">{{ isPlaying ? '⏸ Пауза' : '▶ Слушать' }}</button>
        </div>

        <div class="techniques-list">
          <div v-for="(tech, i) in techniques" :key="i" class="tech-compact-card">
            <div class="tech-left"><span class="t-emoji">{{ tech.emoji }}</span></div>
            <div class="tech-right">
              <div class="d-flex align-items-center gap-2 mb-1">
                <h4 class="m-0 fw-bold">{{ tech.title }}</h4>
                <span class="t-tag">{{ tech.purpose }}</span>
              </div>
              <p class="t-desc"><strong>Что это:</strong> {{ tech.desc }}</p>
              <p class="t-how"><strong>Как делать:</strong> {{ tech.how }}</p>
              <button @click="recordSession(tech.title)" class="mini-record-btn">Завершить практику</button>
            </div>
          </div>
        </div>
      </div>

      <div class="meditation-sidebar">
        <div class="meditation-card history-card">
          <h4 class="fw-bold mb-3" style="font-size: 0.9rem;">История сессий</h4>
          <div v-if="history.length" class="history-list">
            <div v-for="(item, i) in history" :key="i" class="history-item">
              <strong>{{ item.name }}</strong>
              <span>{{ item.date }}</span>
            </div>
          </div>
          <div v-else class="empty-history">🌿 Пока пусто</div>
          <button v-if="history.length" @click="clearHistory" class="clear-btn">Очистить</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
const tracks = ref([
  { name: 'Шум дождя', url: '/rain.mp3', icon: '🌧️', desc: 'Блокирует шум, помогая сосредоточиться на учебе.' },
  { name: 'Звуки леса', url: '/forest.mp3', icon: '🌲', desc: 'Стимулирует творчество и спокойное бодрствование.' },
  { name: 'Дзен-покой', url: '/zen.mp3', icon: '✨', desc: 'Глубокий эмбиент для подготовки нервной системы к отдыху.' },
  { name: 'Шум моря', url: '/sea.mp3', icon: '🌊', desc: 'Ритм волн помогает смыть накопленный за день стресс.' }
]);
const techniques = [
  { title: 'Body Scan', purpose: 'От зажимов', desc: 'Мысленный «просмотр» тела для поиска скрытого напряжения.', how: 'Лягте. Начните с кончиков пальцев ног, медленно «дышите» через каждую зону до макушки.', emoji: '🧘' },
  { title: 'Дыхание 4-7-8', purpose: 'От тревоги', desc: 'Естественный транквилизатор для нервной системы.', how: 'Вдох носом на 4, задержка на 7, выдох на 8. Повторите 4-5 раз.', emoji: '🌬️' },
  { title: 'Осознанность', purpose: 'Для фокуса', desc: 'Практика наблюдения за текущим моментом без оценки.', how: 'Сядьте ровно. Наблюдайте за дыханием. Если мысли улетают, просто возвращайте их назад.', emoji: '🍃' }
];
const currentTrackIndex = ref(0); const isPlaying = ref(false); const volume = ref(0.5); let audio = null;
const setupAudio = () => { if (audio) { audio.pause(); audio = null; }; audio = new Audio(tracks.value[currentTrackIndex.value].url); audio.loop = true; audio.volume = volume.value; }
const toggleMusic = () => { if (!audio) setupAudio(); if (isPlaying.value) { audio.pause(); isPlaying.value = false; } else { audio.play(); isPlaying.value = true; } }
const selectTrack = (i) => { const was = isPlaying.value; currentTrackIndex.value = i; setupAudio(); if (was) audio.play(); }
const updateVolume = () => { if (audio) audio.volume = volume.value }
const history = ref([]);
const recordSession = (n) => {
  history.value.unshift({ name: n, date: new Date().toLocaleString('ru-RU', { day: 'numeric', month: 'short', hour: '2-digit', minute: '2-digit' }) })
  localStorage.setItem('meditationHistory', JSON.stringify(history.value))
}
const clearHistory = () => { history.value = []; localStorage.removeItem('meditationHistory') }
onMounted(() => { const saved = localStorage.getItem('meditationHistory'); if (saved) history.value = JSON.parse(saved) })
onUnmounted(() => { if (audio) audio.pause() })
</script>

<style scoped>
.intro-mini-card { background: white; padding: 20px; border-radius: 25px; margin-bottom: 25px; text-align: left; border: 1px solid #f9f0ff; }
.intro-text h3 { color: var(--primary); font-size: 1.1rem; font-weight: 800; margin-bottom: 4px; }
.intro-text p { color: #888; font-size: 0.85rem; line-height: 1.4; margin: 0; }
.meditation-layout { display: grid; grid-template-columns: 1fr 280px; gap: 20px; align-items: flex-start; }
.meditation-card { background: white; padding: 25px; border-radius: 30px; box-shadow: 0 10px 30px rgba(0,0,0,0.02); }
.player-card { text-align: left; margin-bottom: 25px; border: 2px solid var(--lavender-light); }
.player-header { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 15px; }
.title-row { display: flex; align-items: center; gap: 10px; }
.status-dot { width: 8px; height: 8px; background: #eee; border-radius: 50%; }
.status-dot.is-playing { background: #baffc9; box-shadow: 0 0 8px #baffc9; animation: pulse 2s infinite; }
.track-full-desc { font-size: 0.8rem; color: #999; margin-top: 5px; line-height: 1.3; }
.track-selector { display: flex; gap: 6px; margin-bottom: 15px; flex-wrap: wrap; }
.t-btn { padding: 6px 12px; border-radius: 10px; border: 1px solid #eee; background: #fdfcfd; cursor: pointer; font-size: 0.75rem; font-weight: 700; }
.t-btn.active { background: var(--lavender-light); border-color: var(--primary); color: var(--primary); }
.main-play-btn { width: 100%; padding: 12px; border-radius: 15px; border: none; background: var(--lavender-light); color: var(--primary); font-weight: 800; font-size: 0.9rem; cursor: pointer; }
.techniques-list { display: flex; flex-direction: column; gap: 15px; }
.tech-compact-card { display: flex; gap: 20px; background: white; padding: 20px; border-radius: 25px; text-align: left; border: 1px solid #f9f0ff; }
.t-emoji { font-size: 2rem; background: #fdfaff; width: 50px; height: 50px; display: flex; align-items: center; justify-content: center; border-radius: 15px; }
.tech-right { flex: 1; }
.tech-right h4 { font-size: 1rem; color: #333; margin: 0; }
.t-tag { font-size: 0.65rem; background: var(--lavender-light); color: var(--primary); padding: 2px 8px; border-radius: 6px; font-weight: 800; text-transform: uppercase; }
.t-desc, .t-how { font-size: 0.85rem; color: #666; line-height: 1.4; margin-bottom: 5px; }
.mini-record-btn { background: none; border: 1px solid #eee; color: #aaa; padding: 4px 12px; border-radius: 8px; font-size: 0.75rem; font-weight: 700; cursor: pointer; margin-top: 5px; }
.history-item { display: flex; justify-content: space-between; font-size: 0.75rem; padding-bottom: 5px; border-bottom: 1px solid #f9f9f9; margin-bottom: 5px; }
.clear-btn { margin-top: 15px; background: none; border: none; color: #ccc; cursor: pointer; text-decoration: underline; font-size: 0.7rem; }
.volume-slider { accent-color: var(--primary); width: 60px; height: 4px; }
@keyframes pulse { 0% { opacity: 1; } 50% { opacity: 0.4; } 100% { opacity: 1; } }
</style>