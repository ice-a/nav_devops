<template>
  <Analytics />
  <SpeedInsights />
  <div class="app-container">
    <!-- 玻璃拟态背景层 -->
    <div class="glass-bg"></div>

    <!-- 主要内容区域 -->
    <div class="main-content">
      <!-- 头部区域 -->
      <header class="hero-header">
        <div class="logo-area">
          <div class="logo-icon">🧭</div>
          <h1 class="hero-title">智能导航中心</h1>
        </div>
        <p class="hero-subtitle">探索、发现、连接 - 您的个性化导航助手</p>
      </header>

      <!-- 搜索和过滤区域 -->
      <div class="controls-section">
        <!-- 搜索框 -->
        <SearchBox v-if="!loading" @search="handleSearch" @clear="handleClearSearch"
          :filteredCount="filteredCards.length" />

        <!-- 智能分类标签云 -->
        <div class="tags-cloud" v-if="!loading && !searchQuery">
          <div class="tags-wrapper">
            <button v-for="category in displayCategories" :key="category" class="tag-item" :class="{
              active: selectedCategory === category,
              popular: getCategoryCount(category) > 5,
              trending: Math.random() > 0.7
            }" @click="selectCategory(category)">
              <span class="tag-icon">{{ getCategoryIcon(category) }}</span>
              {{ category }}
              <span class="tag-count">{{ getCategoryCount(category) }}</span>
            </button>
          </div>
        </div>

        <!-- 搜索结果统计 -->
        <div class="result-stats" v-if="searchQuery && filteredCards.length > 0">
          <div class="stats-badge">
            <span class="stats-icon">🎯</span>
            找到 <strong>{{ filteredCards.length }}</strong> 个精准结果
          </div>
        </div>
      </div>

      <!-- 智能内容展示区 -->
      <div class="content-section">
        <!-- 加载状态 -->
        <div class="loading-state" v-if="loading">
          <div class="spinner"></div>
          <p>正在加载导航数据...</p>
        </div>

        <!-- 卡片网格 -->
        <div class="cards-grid" v-else>
          <div v-for="(card, index) in displayCards" :key="card.url + card.index" class="card-wrapper"
            :style="{ animationDelay: `${index * 0.05}s` }">
            <SmartCard :card="card" :index="index" :category-color="getCategoryColor(card.catelog)"
              @click="openLink(card.url)" />
          </div>
        </div>

        <!-- 空状态 -->
        <div class="empty-state" v-if="!loading && displayCards.length === 0">
          <div class="empty-icon">🔍</div>
          <h3>未找到匹配的内容</h3>
          <p>尝试其他关键词或浏览全部分类</p>
          <button class="reset-btn" @click="resetFilters">重置筛选</button>
        </div>
      </div>

      <!-- 底部统计和快捷操作 -->
      <footer class="app-footer">
        <div class="footer-content">
          <div class="stats-overview">
            <div class="stat-item">
              <span class="stat-label">总链接</span>
              <span class="stat-value">{{ allLinks.length }}</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">当前显示</span>
              <span class="stat-value">{{ displayCards.length }}</span>
            </div>
            <div class="stat-item" v-if="searchQuery">
              <span class="stat-label">搜索结果</span>
              <span class="stat-value">{{ filteredCards.length }}</span>
            </div>
            <div class="stat-item" v-else-if="selectedCategory">
              <span class="stat-label">分类</span>
              <span class="stat-value">{{ categoryCards.length }}</span>
            </div>
          </div>

          <div class="quick-actions">
            <button class="action-btn secondary" @click="resetFilters" v-if="searchQuery || selectedCategory">
              <span>✨</span> 重置
            </button>
            <button class="action-btn primary" @click="shuffleDisplay">
              <span>🎲</span> 随机
            </button>
          </div>
        </div>
      </footer>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { Analytics } from '@vercel/analytics/vue';
import { SpeedInsights } from '@vercel/speed-insights/vue';
import SmartCard from './components/SmartCard.vue'
import SearchBox from './components/SearchBox.vue'

const allLinks = ref([])
const loading = ref(true)
const searchQuery = ref('')
const selectedCategory = ref('')

// 搜索功能
const filteredCards = computed(() => {
  if (!searchQuery.value.trim()) return []

  const query = searchQuery.value.toLowerCase()
  return allLinks.value.filter(item => {
    return item.name.toLowerCase().includes(query) ||
      item.desc.toLowerCase().includes(query) ||
      item.catelog.toLowerCase().includes(query) ||
      item.url.toLowerCase().includes(query)
  })
})

// 分类卡片
const categoryCards = computed(() => {
  if (!selectedCategory.value) return []
  return allLinks.value.filter(item => item.catelog === selectedCategory.value)
})

// 分类网格
const displayCategories = computed(() => {
  if (allLinks.value.length === 0) return []
  const categories = [...new Set(allLinks.value.map(item => item.catelog))]
  return categories.sort((a, b) => getCategoryCount(b) - getCategoryCount(a))
})

// 显示的卡片内容
const displayCards = computed(() => {
  if (allLinks.value.length === 0) return []

  let source = []

  if (searchQuery.value.trim() && filteredCards.value.length > 0) {
    source = filteredCards.value
  } else if (selectedCategory.value && categoryCards.value.length > 0) {
    source = categoryCards.value
  } else {
    source = [...allLinks.value].sort(() => Math.random() - 0.5).slice(0, 12)
  }

  return source.map((item, index) => ({
    ...item,
    index
  }))
})

// 分类统计
const getCategoryCount = (category) => {
  return allLinks.value.filter(item => item.catelog === category).length
}

// 分类图标
const getCategoryIcon = (category) => {
  const icons = {
    '影视娱乐': '🎬',
    '个人工具站': '🛠️',
    '编程学习': '📚',
    '前端开发': '💻',
    '开发平台': '🌐',
    '开发工具': '🔧',
    '实用工具': '⚙️',
    '设计资源': '🎨'
  }
  return icons[category] || '📦'
}

// 分类颜色
const getCategoryColor = (category) => {
  const colors = {
    '影视娱乐': '#FF6B6B',
    '个人工具站': '#4ECDC4',
    '编程学习': '#45B7D1',
    '前端开发': '#96CEB4',
    '开发平台': '#FFEAA7',
    '开发工具': '#DDA0DD',
    '实用工具': '#98D8C8',
    '设计资源': '#F7DC6F'
  }
  return colors[category] || '#95A5A6'
}

// 处理搜索
const handleSearch = (query) => {
  searchQuery.value = query
  selectedCategory.value = ''
}

const handleClearSearch = () => {
  searchQuery.value = ''
}

// 选择分类
const selectCategory = (category) => {
  selectedCategory.value = selectedCategory.value === category ? '' : category
  searchQuery.value = ''
}

// 重置筛选
const resetFilters = () => {
  searchQuery.value = ''
  selectedCategory.value = ''
}

// 随机打乱
const shuffleDisplay = () => {
  if (allLinks.value.length > 0) {
    allLinks.value = [...allLinks.value].sort(() => Math.random() - 0.5)
  }
}

// 打开链接
const openLink = (url) => {
  window.open(url, '_blank')
}

// 加载数据
const loadLinks = async () => {
  try {
    const response = await fetch('/link.json')
    if (response.ok) {
      const data = await response.json()
      allLinks.value = data
    } else {
      loadFallbackData()
    }
  } catch (error) {
    loadFallbackData()
  } finally {
    loading.value = false
  }
}

// 内置数据
const loadFallbackData = () => {
  allLinks.value = [
    {
      "name": "爱壹帆影视网",
      "url": "https://www.iyf.lv",
      "catelog": "影视娱乐",
      "desc": "提供影视资源在线观看与下载的影视平台"
    },
    {
      "name": "Kimivod影视资源站",
      "url": "https://kimivod.com/",
      "catelog": "影视娱乐",
      "desc": "聚合各类影视内容的在线播放网站"
    },
    {
      "name": "Umami个人数据统计平台",
      "url": "https://umami.lideshan.top/",
      "catelog": "个人工具站",
      "desc": "基于Umami搭建的轻量级网站访问数据统计分析工具"
    },
    {
      "name": "Docker自建加速站",
      "url": "https://docker.020417.xyz",
      "catelog": "个人工具站",
      "desc": "个人搭建的Docker镜像加速服务平台"
    },
    {
      "name": "在线图片格式转换工具",
      "url": "https://pic.020417.xyz/",
      "catelog": "个人工具站",
      "desc": "支持多种图片格式在线转换的便捷工具"
    },
    {
      "name": "开发者备忘录工具",
      "url": "https://dev.020417.xyz/",
      "catelog": "个人工具站",
      "desc": "面向开发者的在线备忘录与知识整理工具"
    },
    {
      "name": "菜鸟教程网",
      "url": "https://www.runoob.com/",
      "catelog": "编程学习",
      "desc": "提供多编程语言基础教程的入门学习平台"
    },
    {
      "name": "Vue官方文档",
      "url": "https://cn.vuejs.org/",
      "catelog": "前端开发",
      "desc": "渐进式JavaScript框架，适用于构建高效Web应用"
    },
    {
      "name": "MDN Web开发文档",
      "url": "https://developer.mozilla.org/zh-CN/docs/Learn",
      "catelog": "前端开发",
      "desc": "提供Web技术学习资源的权威开发者文档平台"
    },
    {
      "name": "GitHub代码托管平台",
      "url": "https://github.com/",
      "catelog": "开发平台",
      "desc": "全球最大的开源代码托管与协作开发平台"
    },
    {
      "name": "Vercel部署平台",
      "url": "https://vercel.com/",
      "catelog": "前端开发",
      "desc": "专注于Web应用快速构建与部署的平台"
    },
    {
      "name": "Element Plus组件库",
      "url": "https://element-plus.org/zh-CN/",
      "catelog": "前端开发",
      "desc": "基于Vue3的企业级UI组件库"
    },
    {
      "name": "JSON.cn解析工具",
      "url": "https://www.json.cn/",
      "catelog": "开发工具",
      "desc": "轻量级JSON数据解析、格式化与验证工具"
    },
    {
      "name": "ProcessOn在线协作工具",
      "url": "https://www.processon.com/diagrams",
      "catelog": "实用工具",
      "desc": "支持流程图、思维导图的在线协作绘图工具"
    },
    {
      "name": "iLovePDF PDF处理工具",
      "url": "https://www.ilovepdf.com/zh-cn/word_to_pdf",
      "catelog": "实用工具",
      "desc": "支持PDF合并、拆分、转换的在线工具"
    },
    {
      "name": "Carbon代码转图片工具",
      "url": "https://carbon.now.sh/",
      "catelog": "开发工具",
      "desc": "将代码片段转换为精美图片的在线工具"
    },
    {
      "name": "程序员盒子工具平台",
      "url": "https://www.coderutil.com/",
      "catelog": "开发工具",
      "desc": "聚合多种程序员常用工具与技术博文的资源平台"
    },
    {
      "name": "优设设计导航网",
      "url": "https://hao.uisdc.com/",
      "catelog": "设计资源",
      "desc": "为设计师精选优质设计网站与资源的导航平台"
    }
  ]
}

onMounted(() => {
  loadLinks()
})
</script>

<style scoped>
/* 主容器 */
.app-container {
  height: 100vh;
  position: relative;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

/* 玻璃拟态背景层 */
.glass-bg {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background:
    radial-gradient(circle at 10% 20%, rgba(255, 255, 255, 0.1) 0%, transparent 20%),
    radial-gradient(circle at 90% 80%, rgba(255, 255, 255, 0.08) 0%, transparent 20%);
  pointer-events: none;
  z-index: 0;
}

/* 主要内容区域 - 固定布局 */
.main-content {
  position: relative;
  z-index: 1;
  max-width: 1400px;
  margin: 0 auto;
  width: 100%;
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* 头部区域 - 固定高度 */
.hero-header {
  flex-shrink: 0;
  text-align: center;
  padding: 12px 20px;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(20px);
  border-radius: 0 0 20px 20px;
  border: 1px solid rgba(255, 255, 255, 0.25);
  border-bottom: none;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  margin: 0 20px;
}

.logo-area {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  margin-bottom: 6px;
}

.logo-icon {
  font-size: 2rem;
  animation: float 3s ease-in-out infinite;
}

.hero-title {
  font-size: 2.2rem;
  font-weight: 800;
  color: white;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  letter-spacing: -0.5px;
}

.hero-subtitle {
  font-size: 0.95rem;
  color: rgba(255, 255, 255, 0.9);
  font-weight: 400;
  margin-top: 4px;
}

/* 控制区域 */
.controls-section {
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
  gap: 8px;
  align-items: center;
  padding: 0 20px;
}

/* 标签云 */
.tags-cloud {
  width: 100%;
  display: flex;
  justify-content: center;
}

.tags-wrapper {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  justify-content: center;
  max-width: 1000px;
  padding: 4px 0;
}

.tag-item {
  padding: 6px 12px;
  border: none;
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(10px);
  color: white;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 0.85rem;
  position: relative;
  overflow: hidden;
  white-space: nowrap;
}

.tag-item::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
  transition: left 0.5s;
}

.tag-item:hover::before {
  left: 100%;
}

.tag-item:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: translateY(-2px);
}

.tag-item.active {
  background: rgba(255, 255, 255, 0.95);
  color: #667eea;
  transform: scale(1.05) translateY(-2px);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.2);
}

.tag-item.popular {
  background: linear-gradient(135deg, #FF6B6B, #FF8E53);
  font-weight: 700;
}

.tag-item.trending {
  background: linear-gradient(135deg, #FDC830, #F37335);
  animation: pulse 2s ease-in-out infinite;
}

.tag-icon {
  font-size: 1rem;
}

.tag-count {
  background: rgba(0, 0, 0, 0.2);
  padding: 1px 5px;
  border-radius: 8px;
  font-size: 0.7rem;
  font-weight: 700;
}

/* 结果统计 */
.result-stats {
  animation: slideDown 0.3s ease-out;
}

.stats-badge {
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  padding: 12px 24px;
  border-radius: 20px;
  font-size: 0.95rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 8px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.2);
}

.stats-icon {
  font-size: 1.2rem;
}

/* 内容区域 - 可滚动 */
.content-section {
  flex: 1;
  overflow-y: auto;
  overflow-x: hidden;
  min-height: 0;
  padding: 10px;
}

/* 加载状态 */
.loading-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 60px 20px;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(20px);
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 0.25);
  gap: 16px;
}

.spinner {
  width: 50px;
  height: 50px;
  border: 4px solid rgba(255, 255, 255, 0.3);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

.loading-state p {
  color: white;
  font-size: 1.1rem;
  font-weight: 600;
}

/* 卡片网格 */
.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 16px;
  padding: 10px;
}

.card-wrapper {
  animation: fadeInUp 0.5s ease-out both;
}

/* 空状态 */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 60px 20px;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(20px);
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 0.25);
  gap: 12px;
  text-align: center;
}

.empty-icon {
  font-size: 3rem;
  opacity: 0.7;
}

.empty-state h3 {
  color: white;
  font-size: 1.4rem;
  font-weight: 700;
}

.empty-state p {
  color: rgba(255, 255, 255, 0.9);
  font-size: 1rem;
}

.reset-btn {
  margin-top: 8px;
  padding: 10px 20px;
  background: white;
  color: #667eea;
  border: none;
  border-radius: 16px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s;
}

.reset-btn:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.2);
}

/* 底部区域 - 固定在底部 */
.app-footer {
  flex-shrink: 0;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(20px);
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 0.25);
  padding: 16px 20px;
  margin: 0 20px 20px;
}

.footer-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 16px;
}

.stats-overview {
  display: flex;
  gap: 24px;
  flex-wrap: wrap;
}

.stat-item {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.stat-label {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.7);
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.stat-value {
  font-size: 1.3rem;
  color: white;
  font-weight: 800;
}

.quick-actions {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.action-btn {
  padding: 8px 16px;
  border: none;
  border-radius: 12px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.9rem;
}

.action-btn.primary {
  background: white;
  color: #667eea;
}

.action-btn.primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.2);
}

.action-btn.secondary {
  background: rgba(255, 255, 255, 0.2);
  color: white;
}

.action-btn.secondary:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: translateY(-2px);
}

/* 动画定义 */
@keyframes float {

  0%,
  100% {
    transform: translateY(0px);
  }

  50% {
    transform: translateY(-10px);
  }
}

@keyframes pulse {

  0%,
  100% {
    opacity: 1;
  }

  50% {
    opacity: 0.8;
  }
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px) scale(0.95);
  }

  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 响应式设计 */
@media (max-width: 1024px) {
  .cards-grid {
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  }

  .hero-title {
    font-size: 2.4rem;
  }
}

@media (max-width: 768px) {
  .main-content {
    padding: 12px;
    gap: 12px;
  }

  .hero-header {
    padding: 24px 16px 16px;
  }

  .hero-title {
    font-size: 1.8rem;
  }

  .hero-subtitle {
    font-size: 0.95rem;
  }

  .tags-wrapper {
    gap: 6px;
  }

  .tag-item {
    padding: 6px 12px;
    font-size: 0.85rem;
  }

  .cards-grid {
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 12px;
    padding: 6px;
  }

  .footer-content {
    flex-direction: column;
    align-items: stretch;
  }

  .stats-overview {
    justify-content: space-between;
    gap: 12px;
  }

  .quick-actions {
    justify-content: stretch;
  }

  .action-btn {
    flex: 1;
    justify-content: center;
  }
}

@media (max-width: 480px) {
  .hero-title {
    font-size: 1.5rem;
  }

  .logo-icon {
    font-size: 2rem;
  }

  .cards-grid {
    grid-template-columns: 1fr;
  }

  .stats-overview {
    flex-direction: column;
    gap: 8px;
  }

  .stat-item {
    flex-direction: row;
    justify-content: space-between;
    width: 100%;
  }
}
</style>