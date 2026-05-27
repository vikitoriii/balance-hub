<template>
  <div class="habit-page container-fluid py-2">
    <!-- БЛОК ТЕОРИИ -->
    <div class="row mb-4 justify-content-center">
      <div v-for="t in theory" :key="t.title" class="col-md-4 mb-3">
        <div class="card h-100 border-0 shadow-sm rounded-4 p-3 theory-card">
          <h6 class="fw-bold primary-text" style="font-size: 0.9rem;">{{ t.title }}</h6>
          <p class="very-small text-muted m-0">{{ t.desc }}</p>
        </div>
      </div>
    </div>

    <div class="row g-4 justify-content-center">
      <!-- ФОРМА -->
      <div class="col-lg-3">
        <div class="card border-0 shadow-sm p-4 rounded-5 bg-white sticky-top" style="top: 100px;">
          <h4 class="fw-bold mb-3 text-start" style="font-size: 1.1rem;">Новый ритуал</h4>
          <input v-model="newHabitName" type="text" class="form-control custom-input mb-3" placeholder="Название...">
          <button @click="addHabit" class="btn btn-lavender w-100 py-2 rounded-4 fw-bold">Добавить</button>
          
          <div class="mt-4 p-3 rounded-4 bg-light-hint">
            <span class="d-block smaller fw-bold text-primary mb-1">Ваша энергия</span>
            <span class="h4 fw-bold primary-text">✨ {{ totalPoints }} XP</span>
          </div>
        </div>
      </div>

      <!-- ТРЕКЕР -->
      <div class="col-lg-8">
        <div class="d-flex justify-content-between align-items-center mb-4 px-3">
          <div class="text-start">
            <h4 class="fw-bold m-0" style="font-size: 1.1rem;">Ваш прогресс</h4>
            <span class="text-muted smaller">Сегодня: <strong>{{ weekDays[todayIndex] }}</strong> (только этот день доступен для отметки)</span>
          </div>
          <button @click="confirmNewWeek" class="btn btn-outline-lavender btn-sm rounded-pill px-3 fw-bold">🔄 Новая неделя</button>
        </div>

        <div class="habit-list">
          <div v-for="habit in habits" :key="habit.id" class="habit-item-card card border-0 shadow-sm mb-3 rounded-5">
            <div class="card-body p-3">
              <div class="row align-items-center">
                <div class="col-md-4 text-start">
                  <div class="d-flex justify-content-between align-items-center mb-1">
                    <span class="fw-bold m-0" style="font-size: 0.95rem;">{{ habit.name }}</span>
                    <button @click="removeHabit(habit.id)" class="btn-del">×</button>
                  </div>
                  <div class="progress" style="height: 6px; background: #f0f0f0; border-radius: 10px;">
                    <div class="progress-bar" :style="{ width: Math.min((habit.totalDone/21 * 100), 100) + '%', background: 'var(--primary)' }"></div>
                  </div>
                  <span class="smaller text-muted">{{ habit.totalDone }}/21 дн.</span>
                </div>

                <div class="col-md-8">
                  <div class="d-flex justify-content-around align-items-center bg-light p-2 rounded-4">
                    <div v-for="(done, dayIndex) in habit.days" :key="dayIndex" class="day-col text-center">
                      <span class="d-block smaller mb-1 fw-bold" :class="{'text-primary': dayIndex === todayIndex}">{{ weekDays[dayIndex] }}</span>
                      <!-- Валидация: клик работает только если dayIndex === todayIndex -->
                      <div 
                        @click="toggleHabitDay(habit.id, dayIndex)"
                        :class="['day-circle', { 'is-done': done, 'is-disabled': dayIndex !== todayIndex }]"
                      >
                        <span v-if="done">✓</span>
                        <span v-if="dayIndex !== todayIndex && !done" class="lock-icon">🔒</span>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const weekDays = ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс']
const newHabitName = ref('')

// УЗНАЕМ ТЕКУЩИЙ ДЕНЬ (0 - Пн, 6 - Вс)
const todayIndex = computed(() => {
  const day = new Date().getDay(); // 0 - Вс, 1 - Пн ...
  return day === 0 ? 6 : day - 1; 
})

const theory = [
  { title: '🔄 Петля привычки', desc: 'Сигнал -> Действие -> Награда. Придумайте себе маленькое поощрение.' },
  { title: '⏳ Правило 21/90', desc: '21 день для автоматизма, 90 дней для образа жизни.' },
  { title: '💎 2 минуты', desc: 'Начните с того, что занимает 2 минуты. Главное — регулярность.' }
]

const habits = ref([
  { id: 1, name: 'Медитация', totalDone: 0, days: [false, false, false, false, false, false, false] }
])

const totalPoints = computed(() => habits.value.reduce((acc, h) => acc + h.totalDone, 0))

const toggleHabitDay = (id, dayIndex) => {
  // ПРОВЕРКА ПРЕПОДАВАТЕЛЯ: Если день не сегодняшний, запрещаем клик
  if (dayIndex !== todayIndex.value) {
    alert('Вы можете отмечать привычку только за текущий день!');
    return;
  }

  const habit = habits.value.find(h => h.id === id)
  const wasDone = habit.days[dayIndex]
  habit.days[dayIndex] = !habit.days[dayIndex]
  
  if (!wasDone && habit.days[dayIndex]) habit.totalDone++
  else if (wasDone && !habit.days[dayIndex]) habit.totalDone--
  
  saveHabits()
}

const confirmNewWeek = () => {
  if (confirm('Очистить сетку недели? Общий прогресс сохранится.')) {
    habits.value.forEach(h => h.days = [false, false, false, false, false, false, false])
    saveHabits()
  }
}

const addHabit = () => {
  if (newHabitName.value.trim()) {
    habits.value.push({ id: Date.now(), name: newHabitName.value, totalDone: 0, days: [false, false, false, false, false, false, false] })
    newHabitName.value = ''; saveHabits()
  }
}

const removeHabit = (id) => { if(confirm('Удалить?')) { habits.value = habits.value.filter(h => h.id !== id); saveHabits(); } }
const saveHabits = () => localStorage.setItem('habitsDataV4', JSON.stringify(habits.value))
onMounted(() => { const saved = localStorage.getItem('habitsDataV4'); if (saved) habits.value = JSON.parse(saved) })
</script>

<style scoped>
.primary-text { color: var(--primary) !important; }
.smaller { font-size: 0.7rem; }
.very-small { font-size: 0.75rem; line-height: 1.3; }
.bg-light-hint { background: #fdfaff; border: 1px dashed var(--primary); }

.custom-input { border-radius: 12px; border: 1px solid #eee; padding: 10px; font-size: 0.85rem; }
.btn-lavender { background: var(--primary); color: white; border: none; }
.btn-outline-lavender { border: 1px solid var(--lavender-light); color: var(--primary); background: transparent; }

.day-circle {
  width: 30px; height: 30px; border-radius: 8px;
  background: white; border: 1px solid #eee;
  cursor: pointer; display: flex; align-items: center; justify-content: center;
  transition: 0.2s; color: transparent; font-size: 0.8rem;
}
.day-circle.is-done { background: var(--primary); border-color: var(--primary); color: white; }

/* СТИЛИ ДЛЯ ЗАБЛОКИРОВАННЫХ ДНЕЙ */
.day-circle.is-disabled {
  background: #f9f9f9;
  cursor: not-allowed;
  opacity: 0.5;
}
.lock-icon { color: #ccc; font-size: 0.6rem; }

.btn-del { background: none; border: none; color: #eee; font-size: 1.2rem; cursor: pointer; }
.btn-del:hover { color: var(--accent); }
</style>