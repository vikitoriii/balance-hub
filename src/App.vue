<template>
  <div id="app">
    <div class="bg-blob blob-1"></div>
    <div class="bg-blob blob-2"></div>
    <div class="bg-blob blob-3"></div>

    <header>
      <nav>
        <div class="logo">BalanceHub</div>
        <ul>
          <li v-for="tab in menuItems" :key="tab.id">
            <a v-if="currentTab !== tab.id" href="#" @click.prevent="currentTab = tab.id">
              {{ tab.name }}
            </a>
            <span v-else class="active-tab-indicator">{{ tab.name }}</span>
          </li>
        </ul>
      </nav>
    </header>

    <main>
      <h1>
        <template v-if="currentTab === 'Wheel'">колесо жизненного баланса</template>
        <template v-else-if="currentTab === 'Test'">диагностика состояния</template>
        <template v-else-if="currentTab === 'Habits'">трекер полезных привычек</template>
        <template v-else-if="currentTab === 'Meditation'">медитации и покой</template>
        <template v-else-if="currentTab === 'Relax'">антистресс-рум</template>
        <template v-else-if="currentTab === 'Mood'">дневник настроения</template>
        <template v-else-if="currentTab === 'Goals'">мои цели</template>
        <template v-else-if="currentTab === 'Blog'">блог и советы</template>
      </h1>

      <section class="content-container">
        <BalanceWheel v-if="currentTab === 'Wheel'"></BalanceWheel>
        <MentalTest v-else-if="currentTab === 'Test'"></MentalTest>
        <HabitTracker v-else-if="currentTab === 'Habits'"></HabitTracker>
        <MeditationRoom v-else-if="currentTab === 'Meditation'"></MeditationRoom>
        <RelaxRoom v-else-if="currentTab === 'Relax'"></RelaxRoom>
        <MoodDiary v-else-if="currentTab === 'Mood'"></MoodDiary>
        <SmartGoals v-else-if="currentTab === 'Goals'"></SmartGoals>
        <BlogComponent v-else-if="currentTab === 'Blog'"></BlogComponent>
      </section>
    </main>

    <footer>
      <p>&copy; 2024 BalanceHub — Учебная практика</p>
    </footer>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// [Требование №6] Использование фреймворка Vue.js 3
import MeditationRoom from './components/MeditationRoom.vue'
import MentalTest from './components/MentalTest.vue'
import BalanceWheel from './components/BalanceWheel.vue'
import HabitTracker from './components/HabitTracker.vue'
import RelaxRoom from './components/RelaxRoom.vue'
import MoodDiary from './components/MoodDiary.vue'
import SmartGoals from './components/SmartGoals.vue'
import BlogComponent from './components/Blog.vue'

const menuItems = [
  { id: 'Wheel', name: 'Колесо' },
  { id: 'Test', name: 'Тест' },
  { id: 'Habits', name: 'Привычки' },
  { id: 'Meditation', name: 'Медитации' },
  { id: 'Relax', name: 'Антистресс' },
  { id: 'Mood', name: 'Дневник' },
  { id: 'Goals', name: 'Цели' },
  { id: 'Blog', name: 'Блог' }
]

const currentTab = ref('Wheel')
</script>

<style scoped>
/* Только анимации «блобов» для App.vue */
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
  0% { transform: translate(0, 0); }
  100% { transform: translate(50px, 30px); }
}
</style>