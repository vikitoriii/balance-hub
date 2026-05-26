<template>
  <div class="container-fluid py-4">
    <!-- Использование Bootstrap №1: Сетка (Grid System - row, col) -->
    <div class="row justify-content-center">
      
      <!-- ЛЕВАЯ КОЛОНКА: Конструктор -->
      <div class="col-lg-4 mb-4">
        <!-- Использование Bootstrap №2: Карточки (Card) -->
        <div class="card border-0 shadow-sm p-4 rounded-5 bg-white">
          <h3 class="h5 fw-bold mb-4" style="color: var(--primary)">🎯 Поставить цель</h3>
          
          <div class="mb-3 text-start">
            <label class="form-label small fw-bold">Название цели</label>
            <!-- Использование Bootstrap №3: Формы (form-control) -->
            <input v-model="newGoal.title" type="text" class="form-control custom-input" placeholder="Напр: Сдать зачет">
          </div>

          <div class="mb-3 text-start">
            <label class="form-label small fw-bold">Дедлайн</label>
            <!-- Использование JQuery UI №1: Datepicker (Календарь) -->
            <input type="text" id="datepicker" class="form-control custom-input" placeholder="Выберите дату">
          </div>

          <div class="mb-4 text-start">
            <label class="form-label small fw-bold">Описание успеха</label>
            <textarea v-model="newGoal.desc" class="form-control custom-input" rows="3"></textarea>
          </div>

          <button @click="addGoal" class="btn btn-lavender w-100 py-3 rounded-4 fw-bold">
            Сохранить цель
          </button>
          
          <!-- Использование Bootstrap №4: Модальное окно (Modal - вызов через data-bs) -->
          <button class="btn btn-link btn-sm mt-3 text-decoration-none text-muted" data-bs-toggle="modal" data-bs-target="#smartModal">
            Что такое SMART?
          </button>
        </div>
      </div>

      <!-- ПРАВАЯ КОЛОНКА: Список -->
      <div class="col-lg-7">
        <!-- Использование Bootstrap №5: Шкала прогресса (Progress Bar) -->
        <div class="progress-card p-4 rounded-5 mb-4 bg-white shadow-sm">
          <div class="d-flex justify-content-between mb-2">
            <span class="small fw-bold">Общий прогресс целей</span>
            <span class="small fw-bold" :style="{color: 'var(--primary)'}">{{ progressPercentage }}%</span>
          </div>
          <div class="progress" style="height: 10px; background: #f0e6ff;">
            <div class="progress-bar progress-bar-striped progress-bar-animated" 
                 :style="{ width: progressPercentage + '%', backgroundColor: 'var(--primary)' }"></div>
          </div>
        </div>

        <div class="d-flex justify-content-between align-items-center mb-4 px-2">
          <h3 class="h4 fw-bold m-0">Ваш план</h3>
          <span class="text-muted small">Перетаскивайте для смены приоритета</span>
        </div>

        <!-- Использование JQuery UI №2: Sortable (Перетаскивание списка) -->
        <div id="sortable-goals" class="row g-3">
          <div v-for="goal in goals" :key="goal.id" class="col-md-6 goal-item" :data-id="goal.id">
            <div class="card border-0 shadow-sm p-3 h-100 goal-card rounded-5" :class="{ 'completed': goal.completed }">
              <div class="d-flex justify-content-between">
                <h5 class="fw-bold mb-1 text-start" style="font-size: 1rem;">{{ goal.title }}</h5>
                <button @click="removeGoal(goal.id)" class="btn-close" style="font-size: 0.6rem;"></button>
              </div>
              <p class="small text-muted text-start mb-3">{{ goal.desc }}</p>
              <div class="d-flex justify-content-between align-items-center mt-auto pt-2 border-top">
                <span class="badge-date">📅 {{ goal.date }}</span>
                <input type="checkbox" v-model="goal.completed" @change="saveGoals" class="form-check-input custom-check">
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- HTML код для Модального окна Bootstrap -->
    <div class="modal fade" id="smartModal" tabindex="-1" aria-hidden="true">
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content border-0 rounded-5 p-3">
          <div class="modal-header border-0"><h5 class="fw-bold">Методика SMART</h5><button type="button" class="btn-close" data-bs-dismiss="modal"></button></div>
          <div class="modal-body text-start small text-muted">
            SMART — это аббревиатура: Конкретная (Specific), Измеримая (Measurable), Достижимая (Achievable), Значимая (Relevant) и Ограниченная во времени (Time-bound).
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'

const goals = ref([
  { id: 1, title: 'Защитить проект', desc: 'Продемонстрировать работу всех вкладок', date: '03.06.2024', completed: false }
])
const newGoal = ref({ title: '', desc: '' })

// Расчет прогресса для Bootstrap Progress Bar
const progressPercentage = computed(() => {
  if (!goals.value.length) return 0
  const done = goals.value.filter(g => g.completed).length
  return Math.round((done / goals.value.length) * 100)
})

onMounted(() => {
  // ИНИЦИАЛИЗАЦИЯ JQUERY UI Datepicker
  window.$("#datepicker").datepicker({ dateFormat: "dd.mm.yy" });

  // ИНИЦИАЛИЗАЦИЯ JQUERY UI Sortable
  window.$("#sortable-goals").sortable({ opacity: 0.8, cursor: "move" });

  const saved = localStorage.getItem('goalsData')
  if (saved) goals.value = JSON.parse(saved)
})

const addGoal = () => {
  const dateValue = window.$("#datepicker").val()
  if (newGoal.value.title && dateValue) {
    goals.value.unshift({ id: Date.now(), title: newGoal.value.title, desc: newGoal.value.desc, date: dateValue, completed: false })
    newGoal.value = { title: '', desc: '' }; window.$("#datepicker").val(''); saveGoals();
  }
}

const removeGoal = (id) => { goals.value = goals.value.filter(g => g.id !== id); saveGoals(); }
const saveGoals = () => localStorage.setItem('goalsData', JSON.stringify(goals.value))
</script>

<style scoped>
/* Убираем синий цвет Bootstrap и ставим лаванду */
.btn-lavender {
  background-color: var(--primary);
  color: white;
  border: none;
  transition: 0.3s;
}

.btn-lavender:hover {
  background-color: #9a84be;
  color: white;
  transform: translateY(-2px);
}

.custom-input {
  border: 1px solid #f0e6ff;
  background-color: #fdfcfd;
  border-radius: 15px;
  padding: 12px;
}

.custom-input:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 0.25rem rgba(179, 156, 208, 0.15);
}

/* Карточки целей */
.goal-card {
  background: white;
  border: 1px solid #f9f0ff !important;
  transition: 0.3s ease;
  cursor: grab;
}

.goal-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 15px 30px rgba(179, 156, 208, 0.1) !important;
}

.goal-card.is-done {
  background-color: #fcfaff;
  opacity: 0.6;
}

.goal-card.is-done .goal-title {
  text-decoration: line-through;
  color: #aaa;
}

/* Элементы внутри карточки */
.badge-date {
  background: var(--lavender-light);
  color: var(--primary);
  padding: 5px 12px;
  border-radius: 10px;
  font-size: 0.8rem;
  font-weight: 800;
}

.btn-close-custom {
  background: none;
  border: none;
  color: #ddd;
  font-size: 1.5rem;
  line-height: 1;
  transition: 0.3s;
}

.btn-close-custom:hover {
  color: var(--accent);
}

/* Чекбокс лавандового цвета */
.form-check-input:checked {
  background-color: var(--primary);
  border-color: var(--primary);
}

/* Стилизация Progress Bar */
.progress-card {
  background: white;
  padding: 30px;
  border-radius: 35px;
  border: 1px solid #f9f0ff;
}

.progress-percent {
  font-weight: 800;
  color: var(--primary);
}

/* Плейсхолдер при перетаскивании JQ UI */
.sortable-placeholder {
  height: 150px;
  border: 2px dashed var(--lavender-light);
  border-radius: 40px;
  background: transparent;
  visibility: visible !important;
}

@media (max-width: 991px) {
  .goal-constructor { margin-bottom: 30px; }
}
</style>