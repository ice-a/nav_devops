<template>
  <div 
    class="smart-card"
    :class="{ 
      'is-hovered': isHovered,
      'is-pressed': isPressed,
      'is-invalid': isInvalid
    }"
    :style="cardStyle"
    @click="handleClick"
    @mouseenter="handleMouseEnter"
    @mouseleave="handleMouseLeave"
    @mousedown="handleMouseDown"
    @mouseup="handleMouseUp"
  >
    <!-- 卡片背景装饰 - 使用随机渐变色 -->
    <div class="card-decoration" :style="{ background: cardGradient }"></div>
    
    <!-- 卡片内容 -->
    <div class="card-content">
      <div class="card-header">
        <div class="category-badge" :style="{ background: cardGradient }">
          {{ card.catelog }}
        </div>
        <h3 class="card-title">{{ card.name }}</h3>
      </div>
      
      <div class="card-body">
        <p class="card-description">{{ card.desc }}</p>
        <div class="card-url">{{ formatUrl(card.url) }}</div>
      </div>
      
      <div class="card-footer">
        <div class="interaction-hint">
          <span class="hint-icon">🔗</span>
          <span class="hint-text">点击访问</span>
        </div>
        <div class="hover-actions" v-if="isHovered">
          <button class="action-icon" title="新标签打开">
            <span>↗</span>
          </button>
        </div>
      </div>
    </div>

    <!-- 悬停光效层 -->
    <div v-if="isHovered" class="hover-glow" :style="{ background: cardGradient }"></div>
    
    <!-- 点击涟漪效果 -->
    <div v-if="isPressed" class="ripple-effect" :style="{ background: cardGradient }"></div>

    <!-- 无效链接蒙版 -->
    <div v-if="isInvalid" class="invalid-overlay">
      <div class="invalid-content">
        <span class="invalid-icon">✕</span>
        <span class="invalid-text">链接失效</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  card: {
    type: Object,
    required: true
  },
  index: {
    type: Number,
    required: true
  },
  categoryColor: {
    type: String,
    default: '#667eea'
  },
  isInvalid: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['click'])

const isHovered = ref(false)
const isPressed = ref(false)
const rotation = ref(0)

// 预设渐变色方案
const gradientPresets = [
  'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
  'linear-gradient(135deg, #f093fb 0%, #f5576c 100%)',
  'linear-gradient(135deg, #4facfe 0%, #00f2fe 100%)',
  'linear-gradient(135deg, #43e97b 0%, #38f9d7 100%)',
  'linear-gradient(135deg, #fa709a 0%, #fee140 100%)',
  'linear-gradient(135deg, #30cfd0 0%, #330867 100%)',
  'linear-gradient(135deg, #a8edea 0%, #fed6e3 100%)',
  'linear-gradient(135deg, #ff9a9e 0%, #fecfef 100%)',
  'linear-gradient(135deg, #ffecd2 0%, #fcb69f 100%)',
  'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
  'linear-gradient(135deg, #11998e 0%, #38ef7d 100%)',
  'linear-gradient(135deg, #fc466b 0%, #3f5efb 100%)',
  'linear-gradient(135deg, #3f2b96 0%, #a8c0ff 100%)',
  'linear-gradient(135deg, #f857a6 0%, #ff5858 100%)',
  'linear-gradient(135deg, #00b09b 0%, #96c93d 100%)',
  'linear-gradient(135deg, #ffecd2 0%, #fcb69f 100%)',
  'linear-gradient(135deg, #a18cd1 0%, #fbc2eb 100%)',
  'linear-gradient(135deg, #fad0c4 0%, #ffd1ff 100%)'
]

// 根据 URL 生成固定的随机渐变色
const cardGradient = computed(() => {
  let hash = 0
  for (let i = 0; i < props.card.url.length; i++) {
    const char = props.card.url.charCodeAt(i)
    hash = ((hash << 5) - hash) + char
    hash = hash & hash
  }
  const index = Math.abs(hash) % gradientPresets.length
  return gradientPresets[index]
})

// 计算卡片样式
const cardStyle = computed(() => {
  const baseZIndex = isHovered.value ? 100 + props.index : 10 + props.index
  
  let transform = `translateZ(0)`
  
  if (isHovered.value) {
    transform += ` scale(1.03) translateY(-4px)`
  }
  
  if (isPressed.value) {
    transform += ` scale(0.98)`
  }

  return {
    transform,
    zIndex: baseZIndex,
    '--category-color': props.categoryColor
  }
})

// 格式化URL显示
const formatUrl = (url) => {
  try {
    const urlObj = new URL(url)
    return urlObj.hostname.replace('www.', '')
  } catch {
    return url
  }
}

// 鼠标进入
const handleMouseEnter = (event) => {
  isHovered.value = true
  event.stopPropagation()
}

// 鼠标离开
const handleMouseLeave = (event) => {
  isHovered.value = false
  isPressed.value = false
  event.stopPropagation()
}

// 鼠标按下
const handleMouseDown = (event) => {
  isPressed.value = true
  event.stopPropagation()
}

// 鼠标释放
const handleMouseUp = (event) => {
  isPressed.value = false
  event.stopPropagation()
}

// 处理点击
const handleClick = (event) => {
  // 防止在拖动时触发
  if (!isPressed.value) {
    isPressed.value = true
    setTimeout(() => {
      isPressed.value = false
      emit('click')
    }, 100)
  }
}
</script>

<style scoped>
.smart-card {
  position: relative;
  background: rgba(255, 255, 255, 0.98);
  border-radius: 14px;
  padding: 18px;
  min-height: 150px;
  cursor: pointer;
  overflow: hidden;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  border: 1px solid rgba(0, 0, 0, 0.06);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
  backdrop-filter: blur(10px);
  user-select: none;
  transform-style: preserve-3d;
}

.smart-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: var(--category-color);
  opacity: 0.8;
  transition: opacity 0.3s ease;
}

.smart-card:hover::before {
  opacity: 1;
}

.smart-card.is-hovered {
  box-shadow: 0 12px 28px rgba(0, 0, 0, 0.12), 0 0 0 1px rgba(0, 0, 0, 0.05);
  transform: translateY(-4px);
}

.smart-card.is-pressed {
  transform: scale(0.98) translateY(0);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* 背景装饰 */
.card-decoration {
  position: absolute;
  top: 0;
  right: 0;
  width: 60px;
  height: 60px;
  border-radius: 0 16px 0 12px;
  opacity: 0.15;
  transition: opacity 0.3s ease;
  pointer-events: none;
}

.smart-card.is-hovered .card-decoration {
  opacity: 0.25;
  width: 80px;
  height: 80px;
}

/* 卡片内容容器 */
.card-content {
  position: relative;
  z-index: 2;
  height: 100%;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

/* 卡片头部 */
.card-header {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.category-badge {
  display: inline-block;
  padding: 4px 10px;
  border-radius: 12px;
  color: white;
  font-size: 0.7rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  align-self: flex-start;
  backdrop-filter: blur(4px);
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.15);
}

.card-title {
  font-size: 1.1rem;
  font-weight: 700;
  color: #1a1a1a;
  line-height: 1.3;
  margin: 0;
  letter-spacing: -0.2px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* 卡片主体 */
.card-body {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.card-description {
  font-size: 0.88rem;
  color: #555;
  line-height: 1.5;
  margin: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  min-height: 2.6em;
}

.card-url {
  font-size: 0.78rem;
  color: #777;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  padding: 5px 10px;
  background: rgba(0, 0, 0, 0.04);
  border-radius: 8px;
  align-self: flex-start;
}

/* 卡片底部 */
.card-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: auto;
}

.interaction-hint {
  display: flex;
  align-items: center;
  gap: 6px;
  opacity: 0.6;
  transition: opacity 0.3s ease;
  font-size: 0.75rem;
  color: #666;
}

.smart-card.is-hovered .interaction-hint {
  opacity: 1;
  color: var(--category-color);
}

.hint-icon {
  font-size: 0.9rem;
}

.hint-text {
  font-weight: 600;
}

.hover-actions {
  display: flex;
  gap: 4px;
}

.action-icon {
  width: 24px;
  height: 24px;
  border: none;
  background: rgba(0, 0, 0, 0.05);
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.9rem;
  transition: all 0.2s ease;
  color: #666;
}

.action-icon:hover {
  background: var(--category-color);
  color: white;
  transform: scale(1.1);
}

/* 悬停光效层 */
.hover-glow {
  position: absolute;
  top: -4px;
  left: -4px;
  right: -4px;
  bottom: -4px;
  border-radius: 16px;
  opacity: 0;
  animation: glow-pulse 1.5s ease-in-out infinite;
  pointer-events: none;
  z-index: 1;
}

.smart-card.is-hovered .hover-glow {
  opacity: 0.4;
}

@keyframes glow-pulse {
  0%, 100% {
    opacity: 0.3;
    filter: blur(8px);
  }
  50% {
    opacity: 0.6;
    filter: blur(12px);
  }
}

/* 涟漪效果 */
.ripple-effect {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 100px;
  height: 100px;
  border-radius: 50%;
  transform: translate(-50%, -50%) scale(0);
  opacity: 0.6;
  animation: ripple 0.6s ease-out;
  pointer-events: none;
  z-index: 3;
}

@keyframes ripple {
  to {
    transform: translate(-50%, -50%) scale(4);
    opacity: 0;
  }
}

/* 点击反馈 */
.smart-card:active {
  transform: scale(0.97);
}

/* 无效链接蒙版 */
.invalid-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(2px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10;
  border-radius: 14px;
  animation: fadeIn 0.3s ease-out;
}

.invalid-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}

.invalid-icon {
  font-size: 2.5rem;
  color: #ff4444;
  font-weight: bold;
  text-shadow: 0 2px 8px rgba(255, 68, 68, 0.5);
}

.invalid-text {
  color: white;
  font-size: 0.9rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.smart-card.is-invalid {
  cursor: not-allowed;
}

.smart-card.is-invalid .card-content {
  opacity: 0.5;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* 响应式设计 */
@media (max-width: 768px) {
  .smart-card {
    padding: 14px;
    min-height: 130px;
    border-radius: 12px;
  }

  .card-content {
    gap: 10px;
  }

  .card-header {
    gap: 6px;
  }

  .card-title {
    font-size: 1rem;
  }

  .card-description {
    font-size: 0.82rem;
    line-height: 1.4;
  }

  .card-url {
    font-size: 0.72rem;
    padding: 4px 8px;
  }

  .category-badge {
    font-size: 0.65rem;
    padding: 3px 8px;
  }
}

@media (max-width: 480px) {
  .smart-card {
    padding: 12px;
    min-height: 120px;
  }

  .card-title {
    font-size: 0.95rem;
  }

  .card-description {
    font-size: 0.8rem;
    line-height: 1.4;
  }

  .card-url {
    font-size: 0.7rem;
  }
}

/* 深色模式支持（如果需要） */
@media (prefers-color-scheme: dark) {
  .smart-card {
    background: rgba(30, 30, 30, 0.95);
    border-color: rgba(255, 255, 255, 0.1);
  }
  
  .card-title {
    color: #f0f0f0;
  }
  
  .card-description {
    color: #bbb;
  }
  
  .card-url {
    background: rgba(255, 255, 255, 0.05);
    color: #999;
  }
}
</style>