<template>
  <div id="app">
    <!-- Твой живой декоративный фон (сохранен) -->
    <div class="bg-blob blob-1"></div>
    <div class="bg-blob blob-2"></div>
    <div class="bg-blob blob-3"></div>

    <header>
      <nav>
        <div class="logo">BalanceHub</div>
        <ul>
          <li><a href="#" @click.prevent="currentTab = 'Meditation'" :class="{ active: currentTab === 'Meditation' }">Медитации</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Test'" :class="{ active: currentTab === 'Test' }">Тест</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Wheel'" :class="{ active: currentTab === 'Wheel' }">Колесо баланса</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Habits'" :class="{ active: currentTab === 'Habits' }">Привычки</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Relax'" :class="{ active: currentTab === 'Relax' }">Антистресс</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Mood'" :class="{ active: currentTab === 'Mood' }">Дневник</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Goals'" :class="{ active: currentTab === 'Goals' }">Цели</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Blog'" :class="{ active: currentTab === 'Blog' }">Блог</a></li>
        </ul>
      </nav>
    </header>

    <main>
      <!-- Заголовки разделов (все 8 штук) -->
      <h1 v-if="currentTab === 'Meditation'">Медитации и покой</h1>
      <h1 v-else-if="currentTab === 'Test'">Диагностика состояния</h1>
      <h1 v-else-if="currentTab === 'Wheel'">Колесо жизненного баланса</h1>
      <h1 v-else-if="currentTab === 'Habits'">Трекер полезных привычек</h1>
      <h1 v-else-if="currentTab === 'Relax'">Антистресс-рум</h1>
      <h1 v-else-if="currentTab === 'Mood'">Дневник настроения</h1>
      <h1 v-else-if="currentTab === 'Goals'">Мои цели (SMART)</h1>
      <h1 v-else-if="currentTab === 'Blog'">Статьи и советы</h1>

      <section class="content-container">
        <!-- Компоненты (добавлены новые: SmartGoals и Blog) -->
        <MeditationRoom v-if="currentTab === 'Meditation'" />
        <MentalTest v-else-if="currentTab === 'Test'" />
        <BalanceWheel v-else-if="currentTab === 'Wheel'" />
        <HabitTracker v-else-if="currentTab === 'Habits'" />
        <RelaxRoom v-else-if="currentTab === 'Relax'" />
        <MoodDiary v-else-if="currentTab === 'Mood'" />
        <SmartGoals v-else-if="currentTab === 'Goals'" />
        <BlogComponent v-else-if="currentTab === 'Blog'" />
        
        <!-- Заглушка, если вдруг что-то не загрузилось -->
        <div v-else class="placeholder-card">
          <div class="placeholder-icon">🛠️</div>
          <p>Раздел <strong>{{ currentTab }}</strong> в процессе настройки.</p>
        </div>
      </section>
    </main>

    <footer>
      <p>&copy; с заботой о себе </p>
    </footer>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// Импорт всех компонентов (убедись, что все файлы созданы в папке components)
import MeditationRoom from './components/MeditationRoom.vue'
import MentalTest from './components/MentalTest.vue'
import BalanceWheel from './components/BalanceWheel.vue'
import HabitTracker from './components/HabitTracker.vue'
import RelaxRoom from './components/RelaxRoom.vue'
import MoodDiary from './components/MoodDiary.vue'
import SmartGoals from './components/SmartGoals.vue' // Новый
import BlogComponent from './components/Blog.vue'    // Новый

const currentTab = ref('Meditation')
</script>

<style>
/* Твои красивые шрифты и переменные полностью сохранены */
@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@600;700&family=Nunito:wght@400;600;700&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --bg-site: #fefaf9;    
  --primary: #b39cd0;   
  --secondary: #ffcfdf; 
  --lavender-light: #f0e6ff;
  --text-dark: #2d2d2d;
  --white: #ffffff;
}

body {
  background-color: var(--bg-site);
  font-family: 'Nunito', sans-serif;
  color: var(--text-dark);
  min-height: 100vh;
  overflow-x: hidden;
  position: relative;
}

/* Декоративный живой фон (Твои анимации) */
.bg-blob {
  position: fixed;
  border-radius: 50%;
  filter: blur(80px);
  z-index: -1;
  opacity: 0.3;
  animation: float 20s infinite alternate ease-in-out;
}
.blob-1 { width: 400px; height: 400px; background: var(--secondary); top: -100px; left: -100px; }
.blob-2 { width: 500px; height: 500px; background: var(--lavender-light); bottom: -150px; right: -100px; animation-delay: -5s; }
.blob-3 { width: 300px; height: 300px; background: #ffeebb; top: 40%; left: 60%; }

@keyframes float {
  0% { transform: translate(0, 0); }
  100% { transform: translate(80px, 40px); }
}

header {
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(12px);
  padding: 15px 0;
  box-shadow: 0 2px 15px rgba(0,0,0,0.02);
  position: sticky; top: 0; z-index: 1000;
}

nav {
  display: flex; justify-content: space-between; align-items: center;
  max-width: 1400px; margin: 0 auto; padding: 0 30px;
}

.logo {
  font-family: 'Montserrat', sans-serif;
  font-size: 1.6rem; font-weight: 700; color: var(--primary);
  letter-spacing: -0.5px;
}

nav ul { display: flex; list-style: none; gap: 8px; }
nav a {
  text-decoration: none; color: var(--text-dark); font-weight: 700;
  padding: 10px 16px; border-radius: 12px; transition: 0.3s; font-size: 0.9rem;
}
nav a:hover, nav a.active { 
  background: var(--lavender-light); 
  color: var(--primary); 
}

main { 
  max-width: 1400px; 
  margin: 40px auto; 
  padding: 0 30px; 
  min-height: 70vh;
}

h1 { 
  font-family: 'Montserrat', sans-serif; text-align: center; 
  font-size: 2.2rem; margin-bottom: 40px; color: #333; font-weight: 700;
}

.content-container { width: 100%; }

.placeholder-card {
  background: white; padding: 80px 40px; border-radius: 40px; text-align: center;
  box-shadow: 0 10px 30px rgba(0,0,0,0.02); margin: 0 auto; max-width: 600px;
}
.placeholder-icon { font-size: 3rem; margin-bottom: 20px; }

footer { text-align: center; padding: 60px; color: #bbb; font-size: 0.85rem; }

@media (max-width: 1000px) {
  nav { flex-direction: column; gap: 15px; }
  nav ul { flex-wrap: wrap; justify-content: center; }
}
</style>