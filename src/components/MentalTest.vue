<template>
  <div class="test-container">
    <div class="test-card">
      
      <!-- ЭКРАН 1: ИНТРО -->
      <div v-if="step === 'intro'" class="test-intro">
        <div class="test-badge">Методика DASS-21</div>
        <h2>Глубокая диагностика состояния</h2>
        <p>Узнайте свой уровень депрессии, тревоги и стресса. Отвечайте честно, основываясь на ощущениях за <strong>последнюю неделю</strong>.</p>
        <button @click="step = 'quiz'" class="primary-btn">Начать тест</button>
      </div>

      <!-- ЭКРАН 2: ТЕСТ -->
      <div v-else-if="step === 'quiz'" class="test-quiz">
        <div class="progress-container">
          <div class="progress-bar"><div class="progress-fill" :style="{ width: ((currentIndex + 1) / questions.length) * 100 + '%' }"></div></div>
          <span class="step-counter">Вопрос {{ currentIndex + 1 }} из 21</span>
        </div>
        <div class="question-box">
          <h3 class="question-text">{{ questions[currentIndex].text }}</h3>
        </div>
        <div class="options-grid">
          <button v-for="(opt, index) in options" :key="index" @click="handleAnswer(opt.score)" class="option-btn">{{ opt.text }}</button>
        </div>
      </div>

      <!-- ЭКРАН 3: РЕЗУЛЬТАТ -->
      <div v-else-if="step === 'result'" class="test-result">
        <h2 class="result-main-title">Результаты диагностики</h2>
        
        <!-- Краткие карточки по шкалам -->
        <div class="results-grid">
          <div v-for="(val, type) in scores" :key="type" class="result-item" :class="getSeverityClass(val, type)">
            <span class="res-label">{{ type === 'depression' ? 'Депрессия' : type === 'anxiety' ? 'Тревога' : 'Стресс' }}</span>
            <div class="res-val">{{ val }} б.</div>
            <p class="res-lvl">{{ getLevelName(val, type) }}</p>
          </div>
        </div>

        <!-- ТАБЛИЦА РАСШИФРОВКИ -->
        <div class="decoding-section">
          <h4>📊 Как считаются баллы?</h4>
          <div class="decoding-table-wrapper">
            <table class="decoding-table">
              <thead><tr><th>Уровень</th><th>Депр.</th><th>Трев.</th><th>Стресс</th></tr></thead>
              <tbody>
                <tr class="lvl-normal"><td>Норма</td><td>0-9</td><td>0-7</td><td>0-14</td></tr>
                <tr class="lvl-moderate"><td>Умеренный</td><td>14-20</td><td>10-14</td><td>19-25</td></tr>
                <tr class="lvl-extreme"><td>Крайний</td><td>28+</td><td>20+</td><td>34+</td></tr>
              </tbody>
            </table>
          </div>
        </div>

        <!-- ВЕРНУЛИ СОВЕТЫ: ПЕРСОНАЛИЗИРОВАННЫЙ БЛОК -->
        <div class="recommendations-section">
          <h4 class="rec-title">💡 Ваш персональный план в BalanceHub</h4>
          
          <div v-if="recommendations.length > 0" class="rec-list">
            <div v-for="(rec, i) in recommendations" :key="i" class="rec-card">
              <span class="rec-icon">{{ rec.icon }}</span>
              <div class="rec-text">
                <p class="rec-heading">{{ rec.title }}</p>
                <p class="rec-desc">{{ rec.text }}</p>
              </div>
            </div>
          </div>
          <div v-else class="rec-card">
            <span class="rec-icon">✨</span>
            <div class="rec-text">
              <p class="rec-heading">Всё отлично!</p>
              <p class="rec-desc">Ваши показатели в норме. Самое время ставить новые амбициозные цели в разделе "Цели SMART".</p>
            </div>
          </div>
        </div>

        <button @click="restart" class="restart-btn">Пройти заново</button>
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed } from 'vue'

const step = ref('intro')
const currentIndex = ref(0)
const scores = reactive({ depression: 0, anxiety: 0, stress: 0 })

const questions = [
  { text: "Мне было трудно успокоиться", type: "stress" },
  { text: "Я чувствовал сухость во рту", type: "anxiety" },
  { text: "Я не мог испытать никакого положительного чувства", type: "depression" },
  { text: "Я испытывал затруднения при дыхании", type: "anxiety" },
  { text: "Мне было трудно заставить себя что-то сделать", type: "depression" },
  { text: "Я имел тенденцию слишком остро реагировать на ситуации", type: "stress" },
  { text: "Я испытывал дрожь (например, в руках)", type: "anxiety" },
  { text: "Я чувствовал, что трачу много нервной энергии", type: "stress" },
  { text: "Я беспокоился о ситуациях, в которых мог запаниковать", type: "anxiety" },
  { text: "Я чувствовал, что мне нечего ждать в будущем", type: "depression" },
  { text: "Я чувствовал, что становлюсь беспокойным", type: "stress" },
  { text: "Мне было трудно расслабиться", type: "stress" },
  { text: "Я чувствовал уныние и подавленность", type: "depression" },
  { text: "Я нетерпимо относился к вещам, которые мне мешали", type: "stress" },
  { text: "Я чувствовал, что близок к панике", type: "anxiety" },
  { text: "Я не мог ничем воодушевиться", type: "depression" },
  { text: "Я чувствовал, что я мало чего стою как человек", type: "depression" },
  { text: "Я чувствовал себя довольно обидчивым", type: "stress" },
  { text: "Я ощущал сердцебиение без физ. нагрузки", type: "anxiety" },
  { text: "Я чувствовал страх без видимой причины", type: "anxiety" },
  { text: "Я чувствовал, что жизнь бессмысленна", type: "depression" }
]

const options = [
  { text: 'Никогда', score: 0 }, { text: 'Иногда', score: 1 },
  { text: 'Часто', score: 2 }, { text: 'Почти всегда', score: 3 }
]

const handleAnswer = (score) => {
  scores[questions[currentIndex.value].type] += score * 2
  if (currentIndex.value < questions.length - 1) currentIndex.value++
  else step.value = 'result'
}

const getLevelName = (score, type) => {
  const table = {
    depression: [{m:9, n:"Норма"}, {m:13, n:"Легкий"}, {m:20, n:"Умеренный"}, {m:27, n:"Тяжелый"}, {m:99, n:"Крайне тяжелый"}],
    anxiety: [{m:7, n:"Норма"}, {m:9, n:"Легкий"}, {m:14, n:"Умеренный"}, {m:19, n:"Тяжелый"}, {m:99, n:"Крайне тяжелый"}],
    stress: [{m:14, n:"Норма"}, {m:18, n:"Легкий"}, {m:25, n:"Умеренный"}, {m:33, n:"Тяжелый"}, {m:99, n:"Крайне тяжелый"}]
  }
  return table[type].find(l => score <= l.m).n
}

const getSeverityClass = (score, type) => {
  const limits = { depression: 9, anxiety: 7, stress: 14 }
  return score > limits[type] ? 'warning' : 'normal'
}

// ДИНАМИЧЕСКИЕ СОВЕТЫ
const recommendations = computed(() => {
  const recs = []
  if (scores.stress > 14) {
    recs.push({ icon: '🧘', title: 'Снижаем стресс', text: 'Зайдите в "Антистресс-рум". Техника дыхания 4-4 поможет снизить уровень кортизола.' })
  }
  if (scores.anxiety > 7) {
    recs.push({ icon: '⏱️', title: 'Контроль тревоги', text: 'Попробуйте таймер Помодоро. Работа короткими отрезками дает чувство контроля над делами.' })
  }
  if (scores.depression > 9) {
    recs.push({ icon: '📝', title: 'Маленькие шаги', text: 'При депрессивном фоне важны привычки. Отметьте стакан воды в "Привычках" — это уже победа.' })
  }
  return recs
})

const restart = () => {
  step.value = 'intro'; currentIndex.value = 0;
  scores.depression = 0; scores.anxiety = 0; scores.stress = 0;
}
</script>

<style scoped>
.test-container { display: flex; justify-content: center; width: 100%; }
.test-card { background: white; padding: 40px; border-radius: 40px; box-shadow: 0 15px 40px rgba(0,0,0,0.03); max-width: 800px; width: 100%; }
.test-badge { display: inline-block; padding: 6px 15px; background: var(--lavender-light); color: var(--primary); border-radius: 10px; font-size: 0.8rem; font-weight: 800; margin-bottom: 20px; }
.primary-btn { background: var(--primary); color: white; border: none; padding: 16px 40px; border-radius: 18px; font-weight: 800; cursor: pointer; margin-top: 20px; }

/* КВИЗ */
.progress-bar { height: 6px; background: #f0f0f0; border-radius: 10px; margin-bottom: 10px; overflow: hidden; }
.progress-fill { height: 100%; background: var(--primary); transition: 0.4s; }
.step-counter { font-size: 0.8rem; color: #aaa; font-weight: 700; }
.question-box { min-height: 120px; display: flex; align-items: center; justify-content: center; margin: 40px 0; }
.question-text { font-size: 1.5rem; text-align: center; color: #333; }
.options-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; }
.option-btn { padding: 20px; border: 1px solid #eee; border-radius: 20px; background: #fdfcfd; font-family: inherit; font-weight: 700; cursor: pointer; transition: 0.3s; color: #555; }
.option-btn:hover { background: var(--lavender-light); border-color: var(--primary); color: var(--primary); }

/* РЕЗУЛЬТАТЫ */
.results-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 20px; margin: 30px 0; }
.result-item { padding: 20px; border-radius: 25px; border: 1px solid #eee; text-align: center; }
.result-item.normal { border-top: 5px solid #baffc9; }
.result-item.warning { border-top: 5px solid #ffaaa5; background: #fff9f9; }
.res-label { font-weight: 800; color: #999; text-transform: uppercase; font-size: 0.7rem; }
.res-val { font-weight: 900; font-size: 2rem; color: var(--text-dark); margin: 5px 0; }
.res-lvl { font-weight: 700; font-size: 0.9rem; color: var(--primary); }

/* ТАБЛИЦА */
.decoding-section { margin: 30px 0; background: #fcfafc; padding: 20px; border-radius: 25px; }
.decoding-table { width: 100%; border-collapse: collapse; font-size: 0.8rem; text-align: left; }
.decoding-table th { padding: 8px; color: #aaa; border-bottom: 1px solid #eee; }
.decoding-table td { padding: 8px; border-bottom: 1px solid #f5f5f5; }

/* СОВЕТЫ */
.recommendations-section { margin-top: 40px; text-align: left; }
.rec-title { margin-bottom: 20px; color: #333; font-weight: 800; }
.rec-list { display: flex; flex-direction: column; gap: 15px; }
.rec-card { display: flex; gap: 20px; padding: 20px; background: #fdfaff; border-radius: 20px; border: 1px solid var(--lavender-light); align-items: center; }
.rec-icon { font-size: 2rem; }
.rec-heading { font-weight: 800; color: var(--primary); margin-bottom: 4px; }
.rec-desc { font-size: 0.9rem; color: #666; line-height: 1.4; }

.restart-btn { background: none; border: 1px solid #eee; padding: 12px 25px; border-radius: 15px; cursor: pointer; color: #aaa; font-weight: 700; margin-top: 40px; }

@media (max-width: 600px) { .options-grid { grid-template-columns: 1fr; } }
</style>