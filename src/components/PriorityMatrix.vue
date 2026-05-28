<template>
  <div class="matrix-section mt-5 p-4 bg-white rounded-5 shadow-sm">
    <div class="text-center mb-4">
      <h3 class="fw-bold h4 primary-text">🧠 Матрица приоритетов (Эйзенхауэра)</h3>
      <p class="text-muted small">Распределите свои мысли и задачи, чтобы снизить когнитивную нагрузку</p>
    </div>

    <!-- ВВОД НОВОЙ МЫСЛИ/ЗАДАЧИ -->
    <div class="input-group mb-4 shadow-sm rounded-4 overflow-hidden">
      <input 
        v-model="newTaskText" 
        type="text" 
        class="form-control border-0 p-3" 
        placeholder="О чем вы сейчас думаете? (напр: Сдать курсовую)"
        @keyup.enter="addTask"
      >
      <button @click="addTask" class="btn btn-lavender px-4 fw-bold">Добавить</button>
    </div>

    <div class="row g-3">
      <!-- ЗОНЫ СБРОСА (МАТРИЦА 2х2) -->
      <div v-for="zone in zones" :key="zone.id" class="col-md-6">
        <div 
          :id="zone.id" 
          class="matrix-zone p-3 rounded-4 h-100"
          :style="{ background: zone.bgColor, border: '2px dashed ' + zone.color }"
        >
          <div class="d-flex justify-content-between align-items-center mb-2">
            <h5 class="fw-bold m-0" :style="{ color: zone.color }">{{ zone.title }}</h5>
            <span class="badge rounded-pill" :style="{ background: zone.color }">{{ getTasksInZone(zone.id).length }}</span>
          </div>
          <p class="very-small text-muted mb-3">{{ zone.subtitle }}</p>
          
          <!-- Сюда будут падать задачи -->
          <div class="drop-area d-flex flex-wrap gap-2" style="min-height: 80px;">
            <div 
              v-for="task in getTasksInZone(zone.id)" 
              :key="task.id" 
              class="task-sticker shadow-sm"
              :style="{ borderLeft: '4px solid ' + zone.color }"
            >
              {{ task.text }}
              <button @click="removeTask(task.id)" class="btn-close-mini">×</button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- ПУЛ НЕРАСПРЕДЕЛЕННЫХ ЗАДАЧ (Draggable) -->
    <div class="unassigned-zone mt-4 p-3 rounded-4 bg-light">
      <p class="small fw-bold text-muted mb-3">📥 Ваши мысли (перетащите их в нужный квадрат):</p>
      <div class="d-flex flex-wrap gap-2">
        <div 
          v-for="task in getTasksInZone('inbox')" 
          :key="task.id" 
          :id="'task-' + task.id"
          class="task-sticker draggable-task shadow-sm"
        >
          {{ task.text }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick, watch } from 'vue'

const newTaskText = ref('')
const tasks = ref([
  { id: 1, text: 'Подготовить презентацию', zone: 'urgent-important' },
  { id: 2, text: 'Позвонить родителям', zone: 'inbox' },
  { id: 3, text: 'Посмотреть фильм', zone: 'inbox' }
])

const zones = [
  { id: 'urgent-important', title: 'Срочно и Важно', subtitle: 'Сделайте это немедленно', color: '#ffaaa5', bgColor: '#fff5f5' },
  { id: 'not-urgent-important', title: 'Важно, не срочно', subtitle: 'Запланируйте в календаре', color: '#b39cd0', bgColor: '#f9f5ff' },
  { id: 'urgent-not-important', title: 'Срочно, не важно', subtitle: 'Делегируйте или сократите', color: '#fec8a7', bgColor: '#fff9f5' },
  { id: 'not-urgent-not-important', title: 'Не важно, не срочно', subtitle: 'Удалите из списка', color: '#aaa', bgColor: '#f5f5f5' }
]

const getTasksInZone = (zoneId) => tasks.value.filter(t => t.zone === zoneId)

const addTask = () => {
  if (newTaskText.value.trim()) {
    const id = Date.now()
    tasks.value.push({ id, text: newTaskText.value, zone: 'inbox' })
    newTaskText.value = ''
    nextTick(() => initDraggable()) // Ждем появления элемента в DOM
    saveTasks()
  }
}

const removeTask = (id) => {
  tasks.value = tasks.value.filter(t => t.id !== id)
  saveTasks()
}

const saveTasks = () => localStorage.setItem('matrixData', JSON.stringify(tasks.value))

const initDraggable = () => {
  if (window.$ && window.$.fn.draggable) {
    window.$(".draggable-task").draggable({
      revert: "invalid",
      cursor: "grabbing",
      zIndex: 1000,
      start: function() { window.$(this).addClass('grabbing'); },
      stop: function() { window.$(this).removeClass('grabbing'); }
    });
  }
}

const initDroppable = () => {
  zones.forEach(zone => {
    window.$(`#${zone.id}`).droppable({
      accept: ".draggable-task",
      hoverClass: "zone-hover",
      drop: function(event, ui) {
        const taskId = parseInt(ui.helper.attr('id').split('-')[1]);
        const task = tasks.value.find(t => t.id === taskId);
        if (task) {
          task.zone = zone.id; // Меняем зону во Vue
          ui.helper.hide(); // Скрываем клон JQuery
          saveTasks();
        }
      }
    });
  });
}

onMounted(() => {
  const saved = localStorage.getItem('matrixData')
  if (saved) tasks.value = JSON.parse(saved)
  
  nextTick(() => {
    initDraggable();
    initDroppable();
  });
})
</script>

<style scoped>
.matrix-zone { transition: 0.3s ease; min-height: 150px; }
.zone-hover { transform: scale(1.02); filter: brightness(0.98); }

.task-sticker {
  background: white; padding: 8px 12px; border-radius: 12px;
  font-size: 0.85rem; font-weight: 700; color: #444;
  cursor: grab; display: flex; align-items: center; gap: 8px;
  border: 1px solid #eee;
}

.draggable-task { background: var(--lavender-light); border-color: var(--primary); }
.grabbing { cursor: grabbing !important; }

.btn-close-mini {
  background: none; border: none; color: #ccc; font-size: 1.1rem;
  padding: 0; line-height: 1; cursor: pointer;
}
.btn-close-mini:hover { color: var(--accent); }

.very-small { font-size: 0.7rem; line-height: 1.2; }

.border-dashed { border-style: dashed !important; }

.btn-lavender { background: var(--primary); color: white; border: none; transition: 0.3s; }
.btn-lavender:hover { background: #9a84be; }
</style>