<template>
  <div class="habit-section">
    <div class="habit-container">
      
      <!-- ЛЕВАЯ КОЛОНКА -->
      <div class="habit-sidebar">
        <div class="stats-card">
          <h2 class="stats-title">Ваш прогресс</h2>
          <div class="progress-circle">
            <span class="progress-val">{{ completedCount }}/{{ habits.length }}</span>
            <p>привычек выполнено</p>
          </div>
          <div class="progress-bar-bg">
            <div class="progress-bar-fill" :style="{ width: progressPercentage + '%' }"></div>
          </div>
        </div>

        <div class="add-habit-card">
          <h3>Новая привычка</h3>
          <form @submit.prevent="addHabit" class="habit-form">
            <input 
              v-model="newHabitText" 
              type="text" 
              placeholder="Например: Медитация..." 
              maxlength="40"
              required
            >
            <button type="submit" class="add-btn">Добавить</button>
          </form>
        </div>
      </div>

      <!-- ПРАВАЯ КОЛОНКА -->
      <div class="habit-main">
        <div class="filter-bar">
          <button 
            v-for="f in ['Все', 'В процессе', 'Выполнено']" 
            :key="f"
            @click="currentFilter = f"
            :class="['filter-btn', { active: currentFilter === f }]"
          >
            {{ f }}
          </button>
        </div>

        <div class="habit-list">
          <TransitionGroup name="list">
            <div 
              v-for="habit in filteredHabits" 
              :key="habit.id" 
              class="habit-item"
              :class="{ 'is-completed': habit.completed }"
            >
              <label class="checkbox-container">
                <input 
                  type="checkbox" 
                  v-model="habit.completed" 
                  @change="saveHabits"
                >
                <span class="checkmark"></span>
                <span class="habit-text">{{ habit.text }}</span>
              </label>
              
              <button @click="removeHabit(habit.id)" class="delete-btn">&times;</button>
            </div>
          </TransitionGroup>

          <div v-if="filteredHabits.length === 0" class="empty-state">
            <p>В этом списке пока пусто...</p>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const habits = ref([
  
])

const newHabitText = ref('')
const currentFilter = ref('Все')

// ИСПРАВЛЕННАЯ ЛОГИКА ФИЛЬТРАЦИИ
const filteredHabits = computed(() => {
  if (currentFilter.value === 'В процессе') {
    return habits.value.filter(h => !h.completed) // Было !habit.completed, исправила
  }
  if (currentFilter.value === 'Выполнено') {
    return habits.value.filter(h => h.completed)
  }
  return habits.value
})

const completedCount = computed(() => habits.value.filter(h => h.completed).length)
const progressPercentage = computed(() => habits.value.length ? (completedCount.value / habits.value.length) * 100 : 0)

const addHabit = () => {
  if (newHabitText.value.trim()) {
    habits.value.push({ id: Date.now(), text: newHabitText.value, completed: false })
    newHabitText.value = ''
    saveHabits()
  }
}

const removeHabit = (id) => {
  habits.value = habits.value.filter(h => h.id !== id)
  saveHabits()
}

const saveHabits = () => {
  localStorage.setItem('habitsData', JSON.stringify(habits.value))
}

onMounted(() => {
  const saved = localStorage.getItem('habitsData')
  if (saved) habits.value = JSON.parse(saved)
})
</script>

<style scoped>
.habit-container { display: flex; gap: 40px; align-items: flex-start; justify-content: center; flex-wrap: wrap; }
.habit-sidebar { flex: 0 0 320px; display: flex; flex-direction: column; gap: 20px; }
.stats-card { background: white; padding: 25px; border-radius: 30px; text-align: center; box-shadow: 0 10px 30px rgba(0,0,0,0.03); }
.progress-val { font-size: 2.5rem; font-weight: 800; color: var(--primary); }
.progress-bar-bg { height: 8px; background: #f5f5f5; border-radius: 10px; overflow: hidden; margin-top: 15px; }
.progress-bar-fill { height: 100%; background: var(--primary); transition: width 0.5s ease; }

.add-habit-card { background: white; padding: 25px; border-radius: 25px; box-shadow: 0 10px 30px rgba(0,0,0,0.03); }
.habit-form { display: flex; flex-direction: column; gap: 10px; margin-top: 15px; }
.habit-form input { padding: 12px; border-radius: 12px; border: 1px solid #eee; font-family: inherit; outline: none; }
.add-btn { padding: 12px; border-radius: 12px; border: none; background: var(--primary); color: white; font-weight: 700; cursor: pointer; }

.habit-main { flex: 1; max-width: 600px; min-width: 320px; }
.filter-bar { display: flex; gap: 10px; margin-bottom: 20px; }
.filter-btn { padding: 8px 16px; border-radius: 10px; border: 1px solid #eee; background: white; cursor: pointer; color: #888; font-family: inherit; font-size: 0.85rem; }
.filter-btn.active { background: var(--lavender-light); color: var(--primary); border-color: var(--primary); font-weight: 700; }

.habit-list { display: flex; flex-direction: column; gap: 12px; }
.habit-item { background: white; padding: 15px 20px; border-radius: 20px; display: flex; justify-content: space-between; align-items: center; box-shadow: 0 5px 15px rgba(0,0,0,0.02); transition: 0.3s; }
.habit-item.is-completed { opacity: 0.6; }
.habit-item.is-completed .habit-text { text-decoration: line-through; }

.checkbox-container { display: flex; align-items: center; cursor: pointer; gap: 12px; position: relative; }
.checkbox-container input { position: absolute; opacity: 0; }
.checkmark { height: 22px; width: 22px; background-color: #f0f0f0; border-radius: 7px; display: inline-block; position: relative; }
.checkbox-container input:checked ~ .checkmark { background-color: var(--primary); }
.checkmark:after { content: ""; position: absolute; display: none; left: 8px; top: 4px; width: 4px; height: 9px; border: solid white; border-width: 0 2px 2px 0; transform: rotate(45deg); }
.checkbox-container input:checked ~ .checkmark:after { display: block; }
.habit-text { font-weight: 600; color: #444; }

.delete-btn { background: none; border: none; color: #ddd; font-size: 1.4rem; cursor: pointer; }
.delete-btn:hover { color: var(--accent); }

.list-enter-active, .list-leave-active { transition: all 0.3s ease; }
.list-enter-from, .list-leave-to { opacity: 0; transform: translateX(20px); }

.empty-state { text-align: center; padding: 30px; color: #ccc; font-style: italic; }
</style>