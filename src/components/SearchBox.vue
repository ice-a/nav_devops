<template>
  <div class="search-container">
    <div class="search-box" :class="{ 'is-focused': isFocused }">
      <div class="search-icon">
        <span>🔍</span>
      </div>
      
      <input
        v-model="searchQuery"
        type="text"
        placeholder="搜索网站名称、描述、分类或URL..."
        @input="handleSearch"
        @keydown.esc="clearSearch"
        @focus="isFocused = true"
        @blur="isFocused = false"
      />
      
      <button 
        class="clear-btn" 
        v-if="searchQuery" 
        @click="clearSearch"
        title="清空搜索"
      >
        <span>✕</span>
      </button>
      
      <div class="search-decoration"></div>
    </div>
    

    <div class="search-hint" v-if="!searchQuery">
      <span class="hint-icon">💡</span>
      <span class="hint-text">试试搜索 "前端开发" 或 "工具"</span>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  filteredCount: {
    type: Number,
    default: 0
  }
})

const searchQuery = ref('')
const isFocused = ref(false)
const emit = defineEmits(['search', 'clear'])

const handleSearch = () => {
  emit('search', searchQuery.value)
}

const clearSearch = () => {
  searchQuery.value = ''
  emit('clear')
}

// 暴露方法给父组件
defineExpose({
  clearSearch
})
</script>

<style scoped>
.search-container {
  position: relative;
  z-index: 100;
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.search-box {
  position: relative;
  display: flex;
  align-items: center;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 16px;
  padding: 14px 18px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.12);
  backdrop-filter: blur(10px);
  border: 2px solid transparent;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  overflow: hidden;
}

.search-box::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 100%;
  background: linear-gradient(135deg, #667eea, #764ba2);
  opacity: 0;
  transition: opacity 0.3s ease;
  z-index: 0;
}

.search-box.is-focused {
  border-color: #667eea;
  box-shadow: 0 8px 32px rgba(102, 126, 234, 0.35);
  transform: translateY(-2px);
}

.search-box.is-focused::before {
  opacity: 0.05;
}

.search-icon {
  position: relative;
  z-index: 1;
  font-size: 1.3rem;
  margin-right: 12px;
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0.7;
  transition: all 0.3s ease;
}

.search-box.is-focused .search-icon {
  opacity: 1;
  transform: scale(1.1);
}

input {
  flex: 1;
  border: none;
  background: transparent;
  outline: none;
  font-size: 1.05rem;
  padding: 6px 4px;
  color: #333;
  font-weight: 400;
  position: relative;
  z-index: 1;
  transition: all 0.3s ease;
}

input::placeholder {
  color: #999;
  font-style: italic;
  font-weight: 400;
}

.search-box.is-focused input {
  color: #1a1a1a;
}

.clear-btn {
  position: relative;
  z-index: 1;
  width: 28px;
  height: 28px;
  border: none;
  background: rgba(0, 0, 0, 0.05);
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1rem;
  color: #666;
  margin-left: 8px;
  transition: all 0.2s ease;
  font-weight: 600;
}

.clear-btn:hover {
  background: #667eea;
  color: white;
  transform: scale(1.1);
}

.clear-btn:active {
  transform: scale(0.95);
}

/* 搜索装饰 */
.search-decoration {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 0%;
  height: 3px;
  background: linear-gradient(90deg, #667eea, #764ba2, #f093fb);
  transition: width 0.4s ease;
}

.search-box.is-focused .search-decoration {
  width: 100%;
}

/* 搜索统计 */
.search-stats {
  animation: slideDown 0.3s ease-out;
  text-align: center;
}

.stats-content {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  background: rgba(255, 255, 255, 0.95);
  padding: 10px 20px;
  border-radius: 16px;
  font-size: 0.95rem;
  color: #333;
  font-weight: 600;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
}

.stats-icon {
  font-size: 1.1rem;
}

.stats-content strong {
  color: #667eea;
  font-weight: 800;
}

/* 搜索提示 */
.search-hint {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  padding: 6px 14px;
  border-radius: 20px;
  color: white;
  font-size: 0.8rem;
  opacity: 0.8;
  animation: fadeIn 0.5s ease-out;
}

.hint-icon {
  font-size: 1rem;
}

.hint-text {
  font-weight: 500;
}

/* 动画定义 */
@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateY(-8px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 0.9;
  }
}

/* 响应式设计 */
@media (max-width: 768px) {
  .search-container {
    max-width: 100%;
  }

  .search-box {
    padding: 12px 14px;
    border-radius: 14px;
  }

  .search-icon {
    font-size: 1.1rem;
    width: 22px;
    height: 22px;
    margin-right: 10px;
  }

  input {
    font-size: 0.95rem;
  }

  .clear-btn {
    width: 26px;
    height: 26px;
    font-size: 0.9rem;
  }

  .stats-content {
    font-size: 0.85rem;
    padding: 8px 16px;
  }

  .search-hint {
    font-size: 0.78rem;
    padding: 5px 12px;
  }
}

@media (max-width: 480px) {
  .search-box {
    padding: 10px 12px;
    border-radius: 12px;
  }

  input {
    font-size: 0.9rem;
  }

  .search-hint {
    font-size: 0.75rem;
    padding: 4px 10px;
  }
}

/* 深色模式支持 */
@media (prefers-color-scheme: dark) {
  .search-box {
    background: rgba(30, 30, 30, 0.95);
    border-color: rgba(255, 255, 255, 0.1);
  }
  
  .search-box.is-focused {
    border-color: #764ba2;
  }
  
  input {
    color: #f0f0f0;
  }
  
  input::placeholder {
    color: #888;
  }
  
  .clear-btn {
    background: rgba(255, 255, 255, 0.1);
    color: #ccc;
  }
  
  .clear-btn:hover {
    background: #764ba2;
    color: white;
  }
  
  .search-box.is-focused::before {
    opacity: 0.1;
  }
}
</style>