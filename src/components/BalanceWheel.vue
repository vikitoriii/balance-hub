<template>
  <div class="wheel-section">
    <div class="wheel-container">
      
      <div class="controls-card">
        <h2 class="section-subtitle">Ваши показатели</h2>
        
        <div v-for="(sphere, index) in spheres" :key="index" class="slider-group">
          <div class="label-row">
            <div class="name-with-hint">
              <span class="sphere-name">{{ sphere.name }}</span>
              <div class="hint-wrapper">
                <span class="hint-icon">?</span>
                <div class="hint-content">{{ sphere.description }}</div>
              </div>
            </div>
            <span class="sphere-value" :style="{ backgroundColor: sphere.color }">
              {{ sphere.value }}
            </span>
          </div>
          <input type="range" min="0" max="10" v-model.number="sphere.value" @input="saveData" class="custom-slider" :style="{ '--thumb-color': sphere.color }">
        </div>

        <div class="average-compact" :style="{ borderTop: '4px solid ' + interpretation.color }">
          <div class="score-main">
            <span class="avg-text">Средний балл:</span>
            <span class="score-num" :style="{ color: interpretation.color }">{{ averageScore }}</span>
          </div>
          <p class="interpretation-info"><strong>{{ interpretation.title }}:</strong> {{ interpretation.text }}</p>
          
         
          <div class="focus-mini" v-if="weakestSphere && weakestSphere.value < 10">
            <p><strong>Совет по {{ weakestSphere.name }}:</strong> {{ getTip(weakestSphere.name) }}</p>
          </div>
          
          <button @click="resetData" class="reset-btn">Сбросить всё</button>
        </div>
      </div>

      <!-- ПРАВАЯ КОЛОНКА -->
      <div class="visual-area">
        <div class="wheel-card">
          <svg viewBox="0 0 420 420" class="radar-chart">
            <circle v-for="n in 5" :key="n" cx="210" cy="210" :r="n * 38" fill="none" stroke="#f0f0f0" />
            <g v-for="(sphere, i) in spheres" :key="'sector' + i">
              <polygon :points="getSectorPoints(sphere.value, i)" :fill="sphere.color" fill-opacity="0.4" :stroke="sphere.color" stroke-width="2" class="sector-poly" />
            </g>
            <line v-for="(line, i) in axisPoints" :key="'l'+i" :x1="210" :y1="210" :x2="line.x" :y2="line.y" stroke="#eee" stroke-dasharray="3" />
            <text v-for="(point, i) in labelPoints" :key="'t'+i" :x="point.x" :y="point.y" font-size="12" text-anchor="middle" fill="#666" font-weight="700">{{ spheres[i].name }}</text>
          </svg>
        </div>

       
        <div class="priority-card">
          <h3 class="priority-title">🎯 Приоритеты на неделю</h3>
          <div class="priority-list">
            <div v-for="(p, i) in topPriorities" :key="i" class="priority-item">
              <span class="p-number">{{ i + 1 }}</span>
              <div class="p-info">
                <p class="p-name">{{ p.name }}</p>
                <p class="p-desc">{{ getAction(p.name) }}</p>
              </div>
              <div class="p-status" :style="{ background: p.color }"></div>
            </div>
          </div>
        </div>
      </div>

    </div>

    <div class="description-grid">
      <div class="info-tile">
        <h3>🌱 О Колесе баланса</h3>
        <p>Это инструмент для честного разговора с собой. Он помогает увидеть, не забыли ли вы о важных вещах в погоне за результатами.</p>
      </div>
      <div class="info-tile">
        <h3>✨ Маленькие шаги</h3>
        <p>Попробуйте поднять самый низкий показатель всего на 1 балл за следующую неделю. Это проще, чем кажется!</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const spheres = ref([
  { name: 'Здоровье', value: 5, color: '#ffb3ba', description: 'Сон, спорт, питание.' },
  { name: 'Учеба', value: 5, color: '#baffc9', description: 'Знания и личная эффективность.' },
  { name: 'Финансы', value: 5, color: '#bae1ff', description: 'Доходы и финансовая гигиена.' },
  { name: 'Друзья', value: 5, color: '#ffffba', description: 'Общение и поддержка.' },
  { name: 'Семья', value: 5, color: '#ffdfba', description: 'Близкие люди и уют.' },
  { name: 'Отдых', value: 5, color: '#e0bbe4', description: 'Хобби и перезагрузка.' },
  { name: 'Развитие', value: 5, color: '#b39cd0', description: 'Личные цели и навыки.' },
  { name: 'Дух', value: 5, color: '#a2e1db', description: 'Спокойствие и ценности.' }
])

const centerX = 210, centerY = 210, radius = 170

const getSectorPoints = (val, i) => {
  const angleStep = (Math.PI * 2 / 8), aS = angleStep * i - Math.PI / 2, aE = angleStep * (i + 1) - Math.PI / 2, d = (val / 10) * radius
  return `${centerX},${centerY} ${centerX + Math.cos(aS) * d},${centerY + Math.sin(aS) * d} ${centerX + Math.cos(aE) * d},${centerY + Math.sin(aE) * d}`
}
const axisPoints = computed(() => spheres.value.map((_, i) => ({ x: centerX + Math.cos((Math.PI * 2 / 8) * i - Math.PI / 2) * radius, y: centerY + Math.sin((Math.PI * 2 / 8) * i - Math.PI / 2) * radius })))
const labelPoints = computed(() => spheres.value.map((_, i) => ({ x: centerX + Math.cos((Math.PI * 2 / 8) * i - Math.PI / 2 + (Math.PI * 2 / 16)) * (radius + 25), y: centerY + Math.sin((Math.PI * 2 / 8) * i - Math.PI / 2 + (Math.PI * 2 / 16)) * (radius + 25) })))
const averageScore = computed(() => (spheres.value.reduce((acc, s) => acc + s.value, 0) / 8).toFixed(1))

// ЛОГИКА АНАЛИЗА
const topPriorities = computed(() => {
  return [...spheres.value].sort((a, b) => a.value - b.value).slice(0, 3)
})

const getTip = (n) => ({
  'Здоровье': 'пора выспаться.', 'Учеба': 'уделите время книге.', 'Финансы': 'проверьте расходы.', 
  'Друзья': 'напишите другу.', 'Семья': 'позвоните родным.', 'Отдых': 'нужна пауза.',
  'Развитие': 'сделайте шаг к цели.', 'Дух': 'просто подышите.'
}[n])

const getAction = (n) => ({
  'Здоровье': 'Наладить режим сна и добавить прогулку.',
  'Учеба': 'Закрыть один "хвост" или изучить новую тему.',
  'Финансы': 'Составить бюджет на ближайший месяц.',
  'Друзья': 'Запланировать встречу или созвон.',
  'Семья': 'Уделить вечер качественному общению дома.',
  'Отдых': 'Выделить день без соцсетей и работы.',
  'Развитие': 'Пройти урок курса или прочитать главу книги.',
  'Дух': 'Попрактиковать медитацию 10 минут.'
}[n])

const interpretation = computed(() => {
  const s = parseFloat(averageScore.value)
  if (s <= 3.5) return { title: 'Нужна пауза', color: '#ffb3ba', text: 'Энергия на исходе.' }
  if (s <= 6.5) return { title: 'Есть баланс', color: '#ffdfba', text: 'Вы на верном пути.' }
  return { title: 'Гармония', color: '#b39cd0', text: 'Великолепный ритм!' }
})

const saveData = () => localStorage.setItem('balanceData', JSON.stringify(spheres.value))
const resetData = () => { spheres.value.forEach(s => s.value = 5); saveData(); }
onMounted(() => {
  const saved = localStorage.getItem('balanceData'); if (saved) {
    const p = JSON.parse(saved); spheres.value = spheres.value.map((s, i) => ({ ...s, value: p[i].value }))
  }
})
</script>

<style scoped>
.wheel-container { display: flex; gap: 40px; align-items: flex-start; justify-content: center; flex-wrap: wrap; margin-bottom: 50px; }
.controls-card { flex: 0 0 350px; background: white; padding: 30px; border-radius: 35px; box-shadow: 0 10px 30px rgba(0,0,0,0.03); }
.section-subtitle { color: var(--primary); border-left: 5px solid var(--primary); padding-left: 15px; margin-bottom: 25px; font-size: 1.3rem; font-weight: 800; }
.label-row { display: flex; justify-content: space-between; margin-bottom: 8px; font-weight: 700; font-size: 0.95rem; align-items: center; }

.hint-wrapper { position: relative; display: inline-block; margin-left: 5px; }
.hint-icon { width: 16px; height: 16px; background: #f0f0f0; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 10px; cursor: help; color: #aaa; }
.hint-content { position: absolute; z-index: 100; bottom: 25px; left: 0; width: 180px; background: #333; color: #fff; padding: 10px; border-radius: 10px; font-size: 0.75rem; opacity: 0; transition: 0.3s; pointer-events: none; }
.hint-wrapper:hover .hint-content { opacity: 1; }

.sphere-value { padding: 3px 12px; border-radius: 8px; font-size: 0.85rem; color: #000; font-weight: 900; }
.custom-slider { -webkit-appearance: none; width: 100%; height: 6px; background: #f0f0f0; border-radius: 10px; margin-bottom: 18px; }
.custom-slider::-webkit-slider-thumb { -webkit-appearance: none; width: 20px; height: 20px; background: var(--thumb-color); border-radius: 50%; border: 3px solid white; cursor: pointer; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }

.average-compact { border-top: 1px solid #f0f0f0; padding-top: 20px; margin-top: 10px; }
.score-main { display: flex; align-items: center; gap: 10px; margin-bottom: 10px; }
.avg-text { font-size: 0.9rem; color: #888; font-weight: 700; }
.score-num { font-size: 2.2rem; font-weight: 800; }
.interpretation-info { font-size: 0.9rem; margin-bottom: 12px; }
.focus-mini { background: #fdfafc; padding: 12px; border-radius: 15px; margin-bottom: 15px; border-left: 3px solid var(--primary); font-size: 0.85rem; }

/* КАРТОЧКА КОЛЕСА */
.visual-area { flex: 0 1 550px; display: flex; flex-direction: column; gap: 20px; }
.wheel-card { background: white; padding: 30px; border-radius: 40px; box-shadow: 0 10px 30px rgba(0,0,0,0.03); }
.radar-chart { width: 100%; height: auto; overflow: visible; }

/* КАРТОЧКА ПРИОРИТЕТОВ */
.priority-card {
  background: white; padding: 25px; border-radius: 35px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.03);
}
.priority-title { font-size: 1.1rem; color: var(--primary); margin-bottom: 15px; font-weight: 800; text-align: center; }
.priority-list { display: flex; flex-direction: column; gap: 12px; }
.priority-item { 
  display: flex; align-items: center; gap: 15px; 
  padding: 10px; border-radius: 20px; background: #fdfcfd;
}
.p-number { 
  width: 24px; height: 24px; background: var(--lavender-light); 
  border-radius: 50%; display: flex; align-items: center; justify-content: center;
  font-size: 0.8rem; font-weight: 800; color: var(--primary);
}
.p-info { flex: 1; }
.p-name { font-size: 0.9rem; font-weight: 800; color: #444; }
.p-desc { font-size: 0.8rem; color: #888; }
.p-status { width: 8px; height: 8px; border-radius: 50%; }

.description-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 30px; padding-top: 40px; border-top: 1px solid #eee; }
.info-tile { background: white; padding: 30px; border-radius: 30px; box-shadow: 0 5px 20px rgba(0,0,0,0.02); border: 1px solid #f9f9f9; }
.info-tile h3 { color: var(--primary); margin-bottom: 10px; font-size: 1.2rem; }
.info-tile p { font-size: 0.95rem; line-height: 1.6; color: #666; }

.reset-btn { width: 100%; background: #f9f9f9; border: none; padding: 10px; border-radius: 12px; cursor: pointer; color: #ccc; font-size: 0.8rem; font-weight: 700; transition: 0.3s; }
.reset-btn:hover { background: var(--lavender-light); color: var(--primary); }

@media (max-width: 1000px)   { .wheel-container { flex-direction: column; align-items: center; } .visual-area { width: 100%; } }
</style> 