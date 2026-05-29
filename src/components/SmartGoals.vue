<template>
  <div class="container-fluid py-2">
    
    <!-- ТЕОРИЯ SMART -->
    <div class="row mb-5 justify-content-center">
      <div v-for="item in smartTheory" :key="item.letter" class="col-md-4 col-lg-2 mb-2">
        <div class="card h-100 border-0 shadow-sm rounded-4 text-center p-3">
          <div class="h3 fw-bold mb-1" :style="{ color: 'var(--primary)' }">{{ item.letter }}</div>
          <h6 class="fw-bold small">{{ item.title }}</h6>
          <p style="font-size: 0.7rem;" class="text-muted m-0">{{ item.desc }}</p>
        </div>
      </div>
    </div>

    <div class="row g-4 justify-content-center">
      <!-- КОНСТРУКТОР -->
      <div class="col-xl-4 col-lg-5">
        <div class="card border-0 shadow-sm p-4 rounded-5 bg-white">
          <h4 class="fw-bold mb-4 text-start" style="color: var(--primary)">🎯 Новая цель</h4>
          <div class="mb-3 text-start">
            <label class="form-label small fw-bold">Что сделаем?</label>
            <input v-model="newGoal.title" type="text" class="form-control rounded-4 p-3 border-light bg-light" placeholder="Напр: Пройти тест">
          </div>
          <div class="mb-3 text-start">
            <label class="form-label small fw-bold">Дедлайн</label>
            <input type="text" id="goal-date" class="form-control rounded-4 p-3 border-light bg-light" placeholder="Кликните для выбора" readonly>
          </div>
          <button @click="addGoal" class="btn btn-primary w-100 py-3 rounded-4 fw-bold shadow-sm" style="background: var(--primary); border:none;">Поставить цель</button>
        </div>
      </div>

      <!-- СПИСОК -->
      <div class="col-xl-7 col-lg-7">
        <div class="progress-card p-4 rounded-5 mb-4 bg-white shadow-sm border-0">
          <div class="d-flex justify-content-between mb-2 small fw-bold">
            <span>Прогресс целей</span>
            <span>{{ progressPercentage }}%</span>
          </div>
          <div class="progress" style="height: 10px; background: #f3ebff; border-radius: 10px;">
            <div class="progress-bar progress-bar-striped progress-bar-animated" :style="{ width: progressPercentage + '%', backgroundColor: 'var(--primary)' }"></div>
          </div>
        </div>

        <!-- СПИСОК С SORTABLE -->
        <div id="sortable-goals" class="row g-3">
          <div v-for="goal in goals" :key="goal.id" class="col-md-6 goal-card-wrapper" :data-id="goal.id">
            <div class="card border-0 shadow-sm p-4 rounded-5 h-100 goal-card" :class="{ 'is-completed': goal.completed }">
              <div class="d-flex justify-content-between align-items-center mb-3">
                <!-- РУЧКА ⋮⋮ ДЛЯ ПЕРЕТАСКИВАНИЯ -->
                <span class="drag-handle">⋮⋮</span>
                <button @click="removeGoal(goal.id)" class="btn-close shadow-none" style="font-size: 0.7rem;"></button>
              </div>
              
              <h5 class="fw-bold mb-2 text-start" :style="{ textDecoration: goal.completed ? 'line-through' : 'none' }">{{ goal.title }}</h5>
              <p class="small text-muted text-start mb-4 flex-grow-1">{{ goal.desc }}</p>
              
              <div class="d-flex justify-content-between align-items-center pt-3 border-top mt-auto border-light">
                <span class="badge rounded-pill px-3 py-2" style="background: var(--lavender-light); color: var(--primary); font-size: 0.7rem; font-weight: 800;">📅 {{ goal.date }}</span>
                <input type="checkbox" v-model="goal.completed" @change="saveGoals" class="form-check-input shadow-none">
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'

const smartTheory = [
  { letter: 'S', title: 'Specific', desc: 'Конкретная' },
  { letter: 'M', title: 'Measurable', desc: 'Измеримая' },
  { letter: 'A', title: 'Achievable', desc: 'Достижимая' },
  { letter: 'R', title: 'Relevant', desc: 'Важная' },
  { letter: 'T', title: 'Time-bound', desc: 'В срок' }
]

const goals = ref([
  { id: 1, title: 'Защитить практику', desc: 'Показать все разделы проекта', date: '03.06.2024', completed: false }
])

const newGoal = ref({ title: '', desc: '', date: '' })

const progressPercentage = computed(() => {
  if (!goals.value.length) return 0
  return Math.round((goals.value.filter(g => g.completed).length / goals.value.length) * 100)
})

onMounted(() => {
  // Datepicker
  if (window.$ && window.$.fn.datepicker) {
    window.$("#goal-date").datepicker({
      dateFormat: "dd.mm.yy",
      onSelect: function(dateText) { newGoal.value.date = dateText }
    });
  }

  // Sortable с ручкой захвата (handle)
  if (window.$ && window.$.fn.sortable) {
    window.$("#sortable-goals").sortable({
      handle: ".drag-handle", // ТЕПЕРЬ ТАЩИТЬ МОЖНО ТОЛЬКО ЗА ТОЧКИ
      placeholder: "sortable-placeholder col-md-6",
      opacity: 0.8,
      revert: 200,
      tolerance: "pointer"
    });
  }

  const saved = localStorage.getItem('goalsData')
  if (saved) goals.value = JSON.parse(saved)
})

const addGoal = () => {
  const d = window.$("#goal-date").val()
  if (newGoal.value.title && d) {
    goals.value.unshift({ id: Date.now(), title: newGoal.value.title, desc: newGoal.value.desc, date: d, completed: false })
    newGoal.value = { title: '', desc: '', date: '' }; window.$("#goal-date").val(''); saveGoals()
  }
}
const removeGoal = (id) => { goals.value = goals.value.filter(g => g.id !== id); saveGoals(); }
const saveGoals = () => localStorage.setItem('goalsData', JSON.stringify(goals.value))
</script>

<style scoped>
.sortable-placeholder { height: 160px; border: 2px dashed var(--lavender-light); border-radius: 40px; visibility: visible !important; }
.is-completed { opacity: 0.5; }
</style>