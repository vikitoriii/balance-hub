<template>
  <div class="blog-container container-fluid">
    
    <!-- СЕКЦИЯ СТАТЕЙ (Bootstrap Grid & Cards) -->
    <div class="row g-4 mb-5">
      <div class="col-12 text-start">
        <h3 class="fw-bold mb-4">Полезные материалы</h3>
      </div>

      <div v-for="(post, i) in posts" :key="i" class="col-md-6 col-xl-4">
        <article class="card h-100 border-0 shadow-sm rounded-5 overflow-hidden blog-card">
          <div class="card-body p-4 text-start d-flex flex-column">
            <div class="mb-3">
              <span class="badge rounded-pill bg-lavender text-primary px-3 py-2">
                {{ post.tag }}
              </span>
            </div>
            <h4 class="card-title fw-bold h5 mb-3">{{ post.title }}</h4>
            <p class="card-text text-muted small flex-grow-1">{{ post.excerpt }}</p>
            <button class="btn btn-link p-0 text-primary fw-bold text-decoration-none mt-3" @click="openArticle(post)">
              Читать полностью →
            </button>
          </div>
        </article>
      </div>
    </div>

    <!-- ИНТЕРАКТИВНЫЙ FAQ (JQuery UI Accordion) -->
    <div class="row mt-5">
      <div class="col-lg-8 mx-auto text-start">
        <h3 class="fw-bold mb-4 text-center">Часто задаваемые вопросы</h3>
        
        <div id="blog-accordion" class="custom-accordion">
          <h3>Как понять, что мне пора отдохнуть?</h3>
          <div>
            <p>Первые признаки — это раздражительность, нарушение сна и нежелание заниматься тем, что раньше приносило радость. Если вы заметили это, используйте раздел «Антистресс» или «Медитации» прямо сейчас.</p>
          </div>
          
          <h3>Помогает ли Колесо баланса на самом деле?</h3>
          <div>
            <p>Да, это инструмент визуализации. Когда мы видим «перекос» на графике, наш мозг лучше осознает проблему, чем когда мы просто об этом думаем. Это первый шаг к изменениям.</p>
          </div>

          <h3>Сколько времени нужно на формирование привычки?</h3>
          <div>
            <p>Научные данные говорят о сроке от 18 до 254 дней. В среднем — 66 дней. Главное не скорость, а регулярность, которую помогает отслеживать наш трекер.</p>
          </div>

          <h3>Безопасно ли проходить тест DASS-21?</h3>
          <div>
            <p>Это научно признанная методика. Однако помните, что результаты в приложении носят ознакомительный характер. Если баллы очень высокие, лучше обратиться к профессиональному психологу.</p>
          </div>
        </div>
      </div>
    </div>

    <!-- ДЕКОРАТИВНАЯ ЦИТАТА -->
    <div class="row mt-5">
      <div class="col-12">
        <div class="quote-box p-5 rounded-5 text-center italic">
          <p class="fs-4">«Забота о себе — это не эгоизм. Это фундамент, на котором строится ваша жизнь и успех». 🌿</p>
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const posts = ref([
  {
    title: '5 техник борьбы с экзаменационным стрессом',
    tag: 'Учеба',
    excerpt: 'Как сохранять спокойствие, когда до зачета осталась одна ночь, и почему "зубрежка" без перерывов не работает.'
  },
  {
    title: 'Гигиена цифрового пространства',
    tag: 'Ментальное здоровье',
    excerpt: 'Как социальные сети влияют на уровень дофамина и почему важно делать "цифровой детокс" хотя бы раз в неделю.'
  },
  {
    title: 'Искусство маленьких шагов',
    tag: 'Саморазвитие',
    excerpt: 'Почему глобальные цели пугают наш мозг и как обмануть лень, используя систему микро-привычек.'
  }
])

onMounted(() => {
  // Инициализация JQuery UI Accordion
  // window.$ — это обращение к глобальному JQuery
  if (window.$ && window.$.fn.accordion) {
    window.$("#blog-accordion").accordion({
      heightStyle: "content",
      collapsible: true,
      active: false // изначально всё закрыто
    });
  }
})

const openArticle = (post) => {
  alert(`Вы открыли статью: "${post.title}". В полной версии здесь будет переход на лонгрид!`);
}
</script>

<style scoped>
.blog-card {
  transition: all 0.3s ease;
  border: 1px solid #f0f0f0 !important;
}

.blog-card:hover {
  transform: translateY(-10px);
  box-shadow: 0 20px 40px rgba(179, 156, 208, 0.15) !important;
}

.bg-lavender {
  background-color: var(--lavender-light);
}

.text-primary {
  color: var(--primary) !important;
}

/* Стилизация JQuery UI Accordion под наш дизайн */
.custom-accordion :deep(.ui-accordion-header) {
  background: white !important;
  border: 1px solid #eee !important;
  border-radius: 15px !important;
  margin-bottom: 10px !important;
  padding: 15px 20px !important;
  font-family: 'Nunito', sans-serif !important;
  font-weight: 700 !important;
  color: #444 !important;
  outline: none !important;
}

.custom-accordion :deep(.ui-accordion-header-active) {
  border-color: var(--primary) !important;
  color: var(--primary) !important;
}

.custom-accordion :deep(.ui-accordion-content) {
  border: none !important;
  background: transparent !important;
  padding: 10px 20px 20px !important;
  font-size: 0.95rem;
  color: #666;
}

.quote-box {
  background-color: rgba(179, 156, 208, 0.05);
  border: 2px dashed var(--lavender-light);
  color: #888;
  font-style: italic;
}
</style>