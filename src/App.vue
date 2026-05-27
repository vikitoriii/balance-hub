<template>
  <div id="app">
    <!-- Анимации фона сохранены -->
    <div class="bg-blob blob-1"></div>
    <div class="bg-blob blob-2"></div>
    <div class="bg-blob blob-3"></div>

    <header>
      <nav>
        <div class="logo">BalanceHub</div>
        <ul>
          <li><a href="#" @click.prevent="currentTab = 'Meditation'" :class="{ active: currentTab === 'Meditation' }">Медитации</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Test'" :class="{ active: currentTab === 'Test' }">Тест</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Wheel'" :class="{ active: currentTab === 'Wheel' }">Колесо</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Habits'" :class="{ active: currentTab === 'Habits' }">Привычки</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Relax'" :class="{ active: currentTab === 'Relax' }">Антистресс</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Mood'" :class="{ active: currentTab === 'Mood' }">Дневник</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Goals'" :class="{ active: currentTab === 'Goals' }">Цели</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Blog'" :class="{ active: currentTab === 'Blog' }">Блог</a></li>
        </ul>
      </nav>
    </header>

    <main>
      <!-- Уменьшенные заголовки для компактности -->
      <h1 v-if="currentTab === 'Meditation'">Медитации и покой</h1>
      <h1 v-else-if="currentTab === 'Test'">Диагностика состояния</h1>
      <h1 v-else-if="currentTab === 'Wheel'">Колесо баланса</h1>
      <h1 v-else-if="currentTab === 'Habits'">Трекер привычек</h1>
      <h1 v-else-if="currentTab === 'Relax'">Антистресс-рум</h1>
      <h1 v-else-if="currentTab === 'Mood'">Дневник настроения</h1>
      <h1 v-else-if="currentTab === 'Goals'">Мои цели</h1>
      <h1 v-else-if="currentTab === 'Blog'">Блог и советы</h1>

      <section class="content-container">
        <MeditationRoom v-if="currentTab === 'Meditation'" />
        <MentalTest v-else-if="currentTab === 'Test'" />
        <BalanceWheel v-else-if="currentTab === 'Wheel'" />
        <HabitTracker v-else-if="currentTab === 'Habits'" />
        <RelaxRoom v-else-if="currentTab === 'Relax'" />
        <MoodDiary v-else-if="currentTab === 'Mood'" />
        <SmartGoals v-else-if="currentTab === 'Goals'" />
        <BlogComponent v-else-if="currentTab === 'Blog'" />
      </section>
    </main>

    <footer>
      <p>&copy; 2024 BalanceHub — Учебная практика</p>
    </footer>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// Импорт компонентов
import MeditationRoom from './components/MeditationRoom.vue'
import MentalTest from './components/MentalTest.vue'
import BalanceWheel from './components/BalanceWheel.vue'
import HabitTracker from './components/HabitTracker.vue'
import RelaxRoom from './components/RelaxRoom.vue'
import MoodDiary from './components/MoodDiary.vue'
import SmartGoals from './components/SmartGoals.vue'
import BlogComponent from './components/Blog.vue'

const currentTab = ref('Meditation')
</script>

<style scoped>
/* В App.vue оставляем ТОЛЬКО анимации фона. 
   Всё остальное теперь в assets/style.css */

.bg-blob {
  position: fixed;
  border-radius: 50%;
  filter: blur(80px);
  z-index: -1;
  opacity: 0.3;
  animation: float 20s infinite alternate ease-in-out;
}
.blob-1 { width: 350px; height: 350px; background: var(--secondary); top: -100px; left: -100px; }
.blob-2 { width: 450px; height: 450px; background: var(--lavender-light); bottom: -150px; right: -100px; animation-delay: -5s; }
.blob-3 { width: 250px; height: 250px; background: #ffeebb; top: 40%; left: 60%; }

@keyframes float {
  0% { transform: translate(0, 0) scale(1); }
  100% { transform: translate(60px, 30px) scale(1.05); }
}

.content-container {
  width: 100%;
  animation: fadeIn 0.5s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>