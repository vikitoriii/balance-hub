<template>
  <div class="habit-page container-fluid py-2">
    
    <!-- БЛОК ТЕОРИИ (оставляем без изменений) -->
    <div class="row mb-5 justify-content-center">
      <div class="col-12 text-center mb-4">
        <h2 class="fw-bold h4">Искусство маленьких шагов 📈</h2>
        <p class="text-muted small">Как заставить мозг работать на вас, а не против вас.</p>
      </div>
      <div class="col-md-4 mb-3" v-for="t in theory" :key="t.title">
        <div class="card h-100 border-0 shadow-sm rounded-4 p-3 theory-card">
          <h6 class="fw-bold primary-text">{{ t.title }}</h6>
          <p class="very-small text-muted m-0">{{ t.desc }}</p>
        </div>
      </div>
    </div>

    <div class="row g-4 justify-content-center">
      <!-- ЛЕВАЯ КОЛОНКА -->
      <div class="col-lg-3">
        <div class="card border-0 shadow-sm p-4 rounded-5 bg-white sticky-top" style="top: 100px;">
          <h4 class="fw-bold mb-4 text-start">Новый ритуал</h4>
          <div class="mb-3 text-start">
            <label class="form-label small fw-bold">Название</label>
            <input v-model="newHabitName" type="text" class="form-control custom-input" placeholder="Напр: Йога 15 мин">
          </div>
          <button @click="addHabit" class="btn btn-lavender w-100 py-3 rounded-4 fw-bold">Добавить</button>
          
          <div class="mt-4 p-3 rounded-4" style="background: #fdfaff; border: 1px dashed var(--primary)">
            <span class="d-block smaller fw-bold text-primary mb-1">Ваша энергия</span>
            <span class="h3 fw-bold primary-text">✨ {{ totalPoints }} XP</span>
          </div>
        </div>
      </div>

      <!-- ПРАВАЯ КОЛОНКА: ТРЕКЕР -->
      <div class="col-lg-8">
        <div class="d-flex justify-content-between align-items-center mb-4 px-3">
          <div class="text-start">
            <h4 class="fw-bold m-0">Текущий прогресс</h4>
            <span class="text-muted smaller">Отмечайте успехи каждый день</span>
          </div>
          <!-- КНОПКА ПЕРЕКЛЮЧЕНИЯ НЕДЕЛИ -->
          <button @click="confirmNewWeek" class="btn btn-outline-lavender btn-sm rounded-pill px-3 fw-bold">
            🔄 Начать новую неделю
          </button>
        </div>

        <div class="habit-list">
          <div v-for="habit in habits" :key="habit.id" class="habit-item-card card border-0 shadow-sm mb-4 rounded-5">
            <div class="card-body p-4">
              <div class="row align-items-center">
                <div class="col-md-4 text-start mb-3 mb-md-0">
                  <div class="d-flex justify-content-between align-items-center mb-2">
                    <span class="fw-bold h6 m-0">{{ habit.name }}</span>
                    <button @click="removeHabit(habit.id)" class="btn-del">×</button>
                  </div>
                  <div class="maturity-box">
                    <div class="d-flex justify-content-between smaller mb-1">
                      <span>Всего выполнено:</span>
                      <span class="fw-bold">{{ habit.totalDone }} дн.</span>
                    </div>
                    <!-- Шкала на 21 день -->
                    <div class="progress" style="height: 8px; background: #f0f0f0; border-radius: 10px;">
                      <div class="progress-bar progress-bar-striped" 
                           :style="{ width: Math.min((habit.totalDone/21 * 100), 100) + '%', background: 'var(--primary)' }"></div>
                    </div>
                    <span v-if="habit.totalDone >= 21" class="badge bg-success mt-2 w-100">Привычка сформирована! 🎉</span>
                  </div>
                </div>

                <div class="col-md-8">
                  <div class="d-flex justify-content-around align-items-center bg-light p-3 rounded-4">
                    <div v-for="(done, dayIndex) in habit.days" :key="dayIndex" class="day-col text-center">
                      <span class="d-block smaller text-muted mb-2 fw-bold">{{ weekDays[dayIndex] }}</span>
                      <div 
                        @click="toggleHabitDay(habit.id, dayIndex)"
                        :class="['day-circle', { 'is-done': done }]"
                      >
                        <span v-if="done">✓</span>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div v-if="habits.length === 0" class="py-5 text-center opacity-25">
          <p class="fs-4 italic">Добавьте первую привычку, чтобы начать путь 🌿</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const weekDays = ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс']
const newHabitName = ref('')

const theory = [
  { title: '🔄 Петля привычки', desc: 'Сигнал -> Действие -> Награда. Придумайте себе маленькое поощрение за каждый "чек"!' },
  { title: '⏳ Правило 21/90', desc: '21 день для автоматизма, 90 дней для образа жизни. Не делайте пропусков более 2 дней.' },
  { title: '💎 Масштабирование', desc: 'Начните с того, что занимает 2 минуты. Главное — регулярность, а не нагрузка.' }
]

const habits = ref([
  { id: 1, name: 'Медитация', totalDone: 7, days: [false, false, false, false, false, false, false] },
  { id: 2, name: 'Утренняя зарядка', totalDone: 3, days: [false, false, false, false, false, false, false] }
])

const totalPoints = computed(() => habits.value.reduce((acc, h) => acc + h.totalDone, 0))

const toggleHabitDay = (id, dayIndex) => {
  const habit = habits.value.find(h => h.id === id)
  const wasDone = habit.days[dayIndex]
  habit.days[dayIndex] = !habit.days[dayIndex]
  
  if (!wasDone && habit.days[dayIndex]) habit.totalDone++
  else if (wasDone && !habit.days[dayIndex]) habit.totalDone--
  
  saveHabits()
}

// ЛОГИКА НОВОЙ НЕДЕЛИ
const confirmNewWeek = () => {
  if (confirm('Завершить текущую неделю? Все отметки Пн-Вс будут очищены, но общий прогресс сохранится.')) {
    habits.value.forEach(habit => {
      habit.days = [false, false, false, false, false, false, false]
    })
    saveHabits()
  }
}

const addHabit = () => {
  if (newHabitName.value.trim()) {
    habits.value.push({
      id: Date.now(),
      name: newHabitName.value,
      totalDone: 0,
      days: [false, false, false, false, false, false, false]
    })
    newHabitName.value = ''
    saveHabits()
  }
}

const removeHabit = (id) => {
  if(confirm('Удалить привычку?')) {
    habits.value = habits.value.filter(h => h.id !== id)
    saveHabits()
  }
}

const saveHabits = () => localStorage.setItem('habitsDataV3', JSON.stringify(habits.value))

onMounted(() => {
  const saved = localStorage.getItem('habitsDataV3')
  if (saved) habits.value = JSON.parse(saved)
})
</script>

<style scoped>
.primary-text { color: var(--primary) !important; }
.smaller { font-size: 0.7rem; }
.very-small { font-size: 0.8rem; line-height: 1.4; }

.theory-card { border: 1px solid #f0e6ff !important; transition: 0.3s; text-align: left; }
.theory-card:hover { transform: translateY(-5px); background: #fdfaff; }

.custom-input { border-radius: 15px; border: 1px solid #eee; padding: 12px; background: #fdfcfd; font-size: 0.9rem; }
.btn-lavender { background: var(--primary); color: white; border: none; transition: 0.3s; }
.btn-lavender:hover { background: #9a84be; transform: translateY(-2px); }

.btn-outline-lavender { border: 2px solid var(--lavender-light); color: var(--primary); background: transparent; transition: 0.3s; }
.btn-outline-lavender:hover { background: var(--lavender-light); }

.habit-item-card { border: 1px solid #f9f0ff !important; transition: 0.3s; }

.day-circle {
  width: 32px; height: 32px; border-radius: 10px;
  background: white; border: 2px solid #eee;
  cursor: pointer; display: flex; align-items: center; justify-content: center;
  transition: 0.2s; color: transparent; font-weight: 900;
}
.day-circle.is-done {
  background: var(--primary); border-color: var(--primary); color: white;
  box-shadow: 0 4px 10px rgba(179, 156, 208, 0.3);
}

.maturity-box { background: #fdfcfd; padding: 15px; border-radius: 15px; margin-top: 10px; border: 1px solid #f0f0f0; }
.btn-del { background: none; border: none; color: #eee; font-size: 1.2rem; cursor: pointer; transition: 0.3s; }
.btn-del:hover { color: var(--accent); }
</style>