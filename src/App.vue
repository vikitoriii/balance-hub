<template>
  <div id="app">
    <header>
      <nav>
        <div class="logo">BalanceHub</div>
        <!-- Требование №1: Меню в виде списка -->
        <ul>
          <li><a href="#" @click.prevent="currentTab = 'Wheel'">Колесо баланса</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Habits'">Привычки</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Goals'">Цели</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Mood'">Дневник</a></li>
          <li><a href="#" @click.prevent="currentTab = 'Blog'">Блог</a></li>
        </ul>
      </nav>
    </header>

    <!-- Требование: тег main для главного контента -->
    <main>
      <!-- Требование №1: Один заголовок h1 на страницу -->
      <h1 v-if="currentTab === 'Wheel'">Колесо жизненного баланса</h1>
      <h1 v-if="currentTab === 'Habits'">Трекер полезных привычек</h1>
      <h1 v-if="currentTab === 'Goals'">Мои цели (SMART)</h1>
      <h1 v-if="currentTab === 'Mood'">Дневник настроения</h1>
      <h1 v-if="currentTab === 'Blog'">Статьи о ментальном здоровье</h1>

      <section class="content-box">
        <p>Раздел <strong>{{ currentTab }}</strong> находится в разработке.</p>
        <p>Здесь скоро появится интерактивный функционал.</p>
      </section>
    </main>

    <footer>
      <p>&copy; 2024 BalanceHub — Учебная практика</p>
    </footer>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// Создаем реактивную переменную для переключения вкладок
const currentTab = ref('Wheel')
</script>

<style>
/* Базовые стили для пастельной темы и адаптивности */
:root {
  --bg: #fdfaf6;
  --primary: #98ddca;
  --accent: #ffaaa5;
  --text: #4a4a4a;
}

body {
  margin: 0;
  background-color: var(--bg);
  font-family: 'Segoe UI', sans-serif;
  color: var(--text);
}

header {
  background: #fff;
  padding: 15px 30px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.05);
}

nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  max-width: 1100px;
  margin: 0 auto;
}

.logo {
  font-size: 1.6rem;
  font-weight: bold;
  color: var(--primary);
  letter-spacing: 1px;
}

nav ul {
  display: flex;
  list-style: none;
  gap: 25px;
  margin: 0;
  padding: 0;
}

nav a {
  text-decoration: none;
  color: var(--text);
  font-weight: 500;
  transition: 0.3s;
}

nav a:hover {
  color: var(--accent);
}

main {
  max-width: 900px;
  margin: 50px auto;
  padding: 0 20px;
  min-height: 70vh;
}

h1 {
  text-align: center;
  font-size: 2rem;
  margin-bottom: 30px;
}

.content-box {
  background: white;
  padding: 50px;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.03);
  text-align: center;
}

footer {
  text-align: center;
  padding: 30px;
  color: #a0a0a0;
  font-size: 0.9rem;
}

/* Адаптивность (Требование №3) */
@media (max-width: 768px) {
  nav {
    flex-direction: column;
    gap: 15px;
  }
  nav ul {
    gap: 15px;
    font-size: 0.9rem;
    flex-wrap: wrap;
    justify-content: center;
  }
}
</style>