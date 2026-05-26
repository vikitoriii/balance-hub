<template>
  <div class="container-fluid py-2 goals-page">
    
    <!-- БЛОК 1: ТЕОРИЯ SMART (Использование Bootstrap №1: Grid & Cards) -->
    <div class="row mb-5 justify-content-center">
      <div class="col-12 text-center mb-4">
        <h2 class="fw-bold h4 section-header">Методология SMART: от мечты к результату 🧠</h2>
        <p class="text-muted small">Правильно поставленная цель — это 80% успеха</p>
      </div>
      
      <div v-for="item in smartTheory" :key="item.letter" class="col-md-4 col-lg-2 mb-3">
        <div class="card h-100 border-0 shadow-sm rounded-4 text-center p-3 smart-theory-card">
          <div class="display-6 fw-bold mb-1 letter-accent">{{ item.letter }}</div>
          <h6 class="fw-bold small">{{ item.title }}</h6>
          <p class="theory-desc text-muted m-0">{{ item.desc }}</p>
        </div>
      </div>
    </div>

    <div class="row g-4 justify-content-center">
      
      <!-- БЛОК 2: КОНСТРУКТОР ЦЕЛИ (Использование Bootstrap №2: Forms & Validation) -->
      <div class="col-xl-4 col-lg-5">
        <div class="card border-0 shadow-sm p-4 rounded-5 bg-white sticky-top shadow-hover" style="top: 100px; z-index: 10;">
          <h4 class="fw-bold mb-4 text-start primary-text">🎯 Новая цель</h4>
          
          <div class="mb-3 text-start">
            <label class="form-label small fw-bold">Что именно сделать? (Specific)</label>
            <input v-model="newGoal.title" type="text" class="form-control custom-input" placeholder="Напр: Пройти 10 уроков Vue">
          </div>

          <div class="mb-3 text-start">
            <label class="form-label small fw-bold">К какому числу? (Time-bound)</label>
            <!-- Использование JQuery UI №1: Datepicker (ID: goal-date) -->
            <div class="input-group">
              <span class="input-group-text bg-white border-end-0 rounded-start-4">📅</span>
              <input type="text" id="goal-date" class="form-control custom-input border-start-0 rounded-end-4" placeholder="Выбрать дату" readonly>
            </div>
          </div>

          <div class="mb-3 text-start">
            <label class="form-label small fw-bold">Как измерим успех? (Measurable)</label>
            <textarea v-model="newGoal.desc" class="form-control custom-input" rows="2" placeholder="Напр: Получу 100 баллов за тест"></textarea>
          </div>

          <div class="mb-4 text-start">
            <label class="form-label small fw-bold">Приоритет (Bootstrap Badges)</label>
            <select v-model="newGoal.priority" class="form-select custom-input">
              <option value="Высокий">Высокий 🔥</option>
              <option value="Средний">Средний ⚡</option>
              <option value="Низкий">Низкий 🌱</option>
            </select>
          </div>

          <button @click="addGoal" class="btn btn-lavender w-100 py-3 rounded-4 fw-bold shadow-sm">
            Поставить цель
          </button>
          
          <button class="btn btn-link btn-sm mt-3 text-decoration-none text-muted" data-bs-toggle="modal" data-bs-target="#smartHelp">
            ❓ Как проверить цель?
          </button>
        </div>
      </div>

      <!-- БЛОК 3: СПИСОК (Использование JQuery UI №2: Sortable & Bootstrap Progress) -->
      <div class="col-xl-7 col-lg-7">
        
        <!-- Bootstrap Progress Bar №3 -->
        <div class="progress-card p-4 rounded-5 mb-4 bg-white shadow-sm border-0">
          <div class="d-flex justify-content-between mb-2">
            <span class="small fw-bold">Общий прогресс достижений</span>
            <span class="small fw-bold primary-text">{{ progressPercentage }}%</span>
          </div>
          <div class="progress" style="height: 12px; background: #f3ebff; border-radius: 10px;">
            <div class="progress-bar progress-bar-striped progress-bar-animated shadow-none" 
                 :style="{ width: progressPercentage + '%', backgroundColor: 'var(--primary)' }"></div>
          </div>
        </div>

        <div class="d-flex justify-content-between align-items-center mb-4 px-2">
          <h3 class="h5 fw-bold m-0">Активные задачи</h3>
          <span class="text-muted smaller">Перетаскивайте карточки для сортировки</span>
        </div>

        <!-- Контейнер для перетаскивания (JQuery UI Sortable) -->
        <div id="sortable-goals" class="row g-3">
          <div v-for="goal in goals" :key="goal.id" class="col-md-6 goal-card-wrapper" :data-id="goal.id">
            <!-- Использование Bootstrap №4: Cards -->
            <div class="card border-0 shadow-sm p-4 rounded-5 h-100 goal-card" :class="{ 'is-completed': goal.completed }">
              <div class="d-flex justify-content-between align-items-start mb-2">
                <span class="badge-priority" :class="goal.priority.toLowerCase()">{{ goal.priority }}</span>
                <!-- Bootstrap №5: Кнопка закрытия -->
                <button @click="removeGoal(goal.id)" class="btn-close-custom">×</button>
              </div>
              
              <h5 class="fw-bold mb-2 text-start goal-title">{{ goal.title }}</h5>
              <p class="small text-muted text-start mb-4 flex-grow-1">{{ goal.desc }}</p>
              
              <div class="d-flex justify-content-between align-items-center pt-3 border-top mt-auto border-light">
                <span class="badge-date">📅 {{ goal.date }}</span>
                <input type="checkbox" v-model="goal.completed" @change="saveGoals" class="form-check-input custom-check shadow-none">
              </div>
            </div>
          </div>
        </div>

        <div v-if="goals.length === 0" class="py-5 text-center opacity-50">
          <p class="fs-4 italic">Целей пока нет. Начните планировать! ✨</p>
        </div>
      </div>
    </div>

    <!-- МОДАЛЬНОЕ ОКНО (Использование Bootstrap №6: Modal) -->
    <div class="modal fade" id="smartHelp" tabindex="-1" aria-hidden="true">
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content border-0 rounded-5 p-4 shadow-lg">
          <div class="modal-header border-0"><h5 class="fw-bold">Чек-лист SMART</h5><button type="button" class="btn-close" data-bs-dismiss="modal"></button></div>
          <div class="modal-body text-start small">
            <p>Чтобы цель была достигнута, проверьте её:</p>
            <ul class="list-unstyled">
              <li class="mb-2">📍 <strong>S:</strong> Цель конкретна? (Что именно?)</li>
              <li class="mb-2">📊 <strong>M:</strong> Есть ли цифра успеха? (Сколько?)</li>
              <li class="mb-2">🧗 <strong>A:</strong> Это реально выполнить за этот срок?</li>
              <li class="mb-2">💎 <strong>R:</strong> Вам это правда нужно сейчас?</li>
              <li class="mb-2">⏰ <strong>T:</strong> Есть ли финальная дата?</li>
            </ul>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'

// Теория SMART
const smartTheory = [
  { letter: 'S', title: 'Specific', desc: 'Конкретная' },
  { letter: 'M', title: 'Measurable', desc: 'Измеримая' },
  { letter: 'A', title: 'Achievable', desc: 'Достижимая' },
  { letter: 'R', title: 'Relevant', desc: 'Значимая' },
  { letter: 'T', title: 'Time-bound', desc: 'В срок' }
]

const goals = ref([
  { id: 1, title: 'Защитить учебную практику', desc: 'Продемонстрировать 8 разделов и идеальный код', date: '03.06.2024', priority: 'Высокий', completed: false },
  { id: 2, title: 'Утренняя медитация', desc: 'Провести сессию в разделе Медитации 10 минут', date: '31.05.2024', priority: 'Средний', completed: true }
])

const newGoal = ref({ title: '', desc: '', date: '', priority: 'Средний' })

// Реактивный расчет прогресса (Требование №4: DOM/Data Processing)
const progressPercentage = computed(() => {
  if (!goals.value.length) return 0
  const completed = goals.value.filter(g => g.completed).length
  return Math.round((completed / goals.value.length) * 100)
})

onMounted(() => {
  // 1. Инициализация JQuery UI Datepicker (Требование №9)
  if (window.$ && window.$.fn.datepicker) {
    window.$("#goal-date").datepicker({
      dateFormat: "dd.mm.yy",
      monthNames: ["Январь","Февраль","Март","Апрель","Май","Июнь","Июль","Август","Сентябрь","Октябрь","Ноябрь","Декабрь"],
      dayNamesMin: ["Вс","Пн","Вт","Ср","Чт","Пт","Сб"],
      onSelect: function(dateText) {
        newGoal.value.date = dateText
      }
    });
  }

  // 2. Инициализация JQuery UI Sortable (Требование №9)
  if (window.$ && window.$.fn.sortable) {
    window.$("#sortable-goals").sortable({
      placeholder: "sortable-placeholder col-md-6",
      opacity: 0.8,
      cursor: "move",
      revert: 200
    });
  }

  const saved = localStorage.getItem('goalsData')
  if (saved) goals.value = JSON.parse(saved)
})

const addGoal = () => {
  const dateFromJQ = window.$("#goal-date").val()
  if (newGoal.value.title && dateFromJQ) {
    goals.value.unshift({
      id: Date.now(),
      title: newGoal.value.title,
      desc: newGoal.value.desc,
      date: dateFromJQ,
      priority: newGoal.value.priority,
      completed: false
    })
    // Сброс формы
    newGoal.value = { title: '', desc: '', date: '', priority: 'Средний' }
    window.$("#goal-date").val('')
    saveGoals()
  } else {
    alert("Пожалуйста, заполните название и выберите дату!")
  }
}

const removeGoal = (id) => {
  goals.value = goals.value.filter(g => g.id !== id)
  saveGoals()
}

const saveGoals = () => {
  localStorage.setItem('goalsData', JSON.stringify(goals.value))
}
</script>

<style scoped>
/* ПЕРСОНАЛИЗАЦИЯ (НИКАКОГО СИНЕГО) */
.primary-text { color: var(--primary) !important; }
.letter-accent { color: var(--primary); text-shadow: 2px 2px 0px var(--lavender-light); }

.btn-lavender { 
  background-color: var(--primary); 
  color: white; 
  border: none; 
  transition: 0.3s cubic-bezier(0.4, 0, 0.2, 1); 
}
.btn-lavender:hover { 
  background-color: #9a84be; 
  color: white; 
  transform: translateY(-2px); 
  box-shadow: 0 8px 20px rgba(179, 156, 208, 0.3);
}

.custom-input { 
  border: 1px solid #f0e6ff; 
  background-color: #fdfcfd; 
  border-radius: 15px; 
  padding: 12px; 
  font-size: 0.9rem;
}
.custom-input:focus { 
  border-color: var(--primary); 
  box-shadow: 0 0 0 0.25rem rgba(179, 156, 208, 0.15); 
}

/* КАРТОЧКИ ТЕОРИИ */
.smart-theory-card { 
  transition: 0.3s; 
  border: 1px solid #f9f0ff !important; 
}
.smart-theory-card:hover { 
  transform: translateY(-5px); 
  background: var(--lavender-light); 
}
.theory-desc { font-size: 0.75rem; line-height: 1.3; }

/* КАРТОЧКИ ЦЕЛЕЙ */
.goal-card { 
  background: white; 
  border: 1px solid #f9f0ff !important; 
  cursor: grab; 
  transition: 0.3s ease; 
}
.goal-card:hover { 
  transform: translateY(-5px); 
  box-shadow: 0 15px 30px rgba(179, 156, 208, 0.1) !important; 
}
.goal-card.is-completed { 
  opacity: 0.6; 
  background: #fdfcfd !important; 
}
.goal-card.is-completed .goal-title { 
  text-decoration: line-through; 
  color: #aaa; 
}

/* ПРИОРИТЕТЫ */
.badge-priority { 
  font-size: 0.65rem; 
  text-transform: uppercase; 
  font-weight: 800; 
  padding: 4px 12px; 
  border-radius: 8px; 
}
.высокий { background: #fff1f0; color: #ff7875; border: 1px solid #ffa39e; }
.средний { background: #fff7e6; color: #ffa940; border: 1px solid #ffd591; }
.низкий { background: #f6ffed; color: #73d13d; border: 1px solid #b7eb8f; }

.badge-date { 
  background: var(--lavender-light); 
  color: var(--primary); 
  padding: 6px 14px; 
  border-radius: 12px; 
  font-size: 0.75rem; 
  font-weight: 800; 
}

.btn-close-custom { 
  background: none; border: none; color: #ddd; 
  font-size: 1.5rem; transition: 0.3s; line-height: 1; 
}
.btn-close-custom:hover { color: var(--accent); }

.custom-check:checked { background-color: var(--primary); border-color: var(--primary); }

.progress-card { border: 1px solid #f9f0ff; }

/* Кастомизация JQuery UI Datepicker */
:global(.ui-datepicker) {
  z-index: 9999 !important;
  border: none !important;
  border-radius: 20px !important;
  box-shadow: 0 15px 40px rgba(0,0,0,0.12) !important;
  padding: 15px !important;
  font-family: 'Nunito', sans-serif !important;
}
:global(.ui-datepicker-header) {
  background: var(--primary) !important;
  border: none !important;
  border-radius: 12px !important;
  color: white !important;
}

/* Стили плейсхолдера Sortable */
.sortable-placeholder { 
  height: 160px; 
  border: 2px dashed var(--lavender-light); 
  border-radius: 40px; 
  visibility: visible !important; 
  background: transparent;
}
</style>