<template>
  <div class="blog-container container-fluid">
    
    <!-- СПИСОК СТАТЕЙ -->
    <div v-if="!selectedPost" class="blog-list-view animate-in">
      <div class="row g-4 mb-5 text-start px-2">
        <div class="col-12">
          <h3 class="fw-bold mb-1">Библиотека BalanceHub</h3>
          <p class="text-muted small mb-4">9 экспертных лонгридов для вашего развития</p>
        </div>

        <div v-for="post in blogPosts" :key="post.id" class="col-md-6 col-lg-4">
          <article class="card h-100 border-0 shadow-sm rounded-5 blog-card">
            <div class="card-body p-4 d-flex flex-column">
              <div class="mb-3">
                <span class="custom-badge">{{ post.tag }}</span>
              </div>
              <h4 class="card-title fw-bold h6 mb-3">{{ post.title }}</h4>
              <p class="card-text text-muted small flex-grow-1">{{ post.excerpt }}</p>
              <button class="btn-read-more" @click="openArticle(post)">
                Читать полностью →
              </button>
            </div>
          </article>
        </div>
      </div>
      
      <!-- ПОДКЛЮЧАЕМ НОВЫЙ ИНТЕРАКТИВНЫЙ ИНСТРУМЕНТ (ВМЕСТО АККОРДЕОНА) -->
      <StickerJar />
    </div>

    <!-- РЕЖИМ ЧТЕНИЯ СТАТЬИ -->
    <div v-else class="article-full-view text-start animate-in">
      <button @click="selectedPost = null" class="btn-back mb-4">← К списку материалов</button>
      
      <div class="article-header mb-4">
        <span class="custom-badge mb-3 d-inline-block">{{ selectedPost.tag }}</span>
        <h2 class="fw-bold h3">{{ selectedPost.title }}</h2>
      </div>

      <div class="article-content card border-0 shadow-sm p-4 p-md-5 rounded-5">
        <div v-html="selectedPost.fullContent" class="long-text-body"></div>
        <div class="article-footer mt-5 p-4 rounded-4" style="background: var(--bg-site)">
          <p class="small m-0 text-muted italic">Материал подготовлен специально для проекта BalanceHub.</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { blogPosts } from '../data/blogContent.js'


const selectedPost = ref(null)

const openArticle = (post) => {
  selectedPost.value = post;
  window.scrollTo(0, 0);
}
</script>

<style scoped>
.animate-in { animation: fadeIn 0.4s ease-out; }
@keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

.custom-badge { background-color: var(--lavender-light) !important; color: var(--primary) !important; padding: 5px 12px; border-radius: 8px; font-size: 0.65rem; font-weight: 800; text-transform: uppercase; }
.btn-read-more { background: none; border: none; color: var(--primary); font-weight: 800; font-size: 0.8rem; padding: 0; margin-top: 15px; text-align: left; transition: 0.3s; }
.btn-read-more:hover { opacity: 0.7; transform: translateX(5px); }
.blog-card { border: 1px solid #f9f0ff !important; transition: 0.3s; height: 100%; }
.blog-card:hover { transform: translateY(-5px); box-shadow: 0 10px 25px rgba(179, 156, 208, 0.1) !important; }

.btn-back { background: var(--white); border: 1px solid #eee; padding: 8px 16px; border-radius: 10px; color: var(--primary); font-weight: 800; cursor: pointer; font-size: 0.8rem; }
.long-text-body { line-height: 1.8; color: #444; }
.long-text-body :deep(h5) { margin-top: 30px; font-weight: 800; color: var(--primary); border-left: 3px solid var(--secondary); padding-left: 15px; }
.long-text-body :deep(p) { margin-bottom: 20px; font-size: 1rem; }
.long-text-body :deep(ul) { margin-bottom: 20px; }
.long-text-body :deep(li) { margin-bottom: 10px; }
</style>