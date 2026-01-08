<template>
  <div
    class="floating-card"
    :class="{ 'is-hovered': isHovered }"
    :style="cardStyle"
    @click="handleClick"
    @mouseenter="handleMouseEnter"
    @mouseleave="handleMouseLeave"
  >
    <div class="card-content">
      <div class="card-title">{{ card.name }}</div>
      <div class="card-url">{{ card.url }}</div>
      <div class="card-desc">{{ card.desc }}</div>
      <div class="card-category">{{ card.catelog }}</div>
    </div>
    <!-- 悬停时的发光边框效果 -->
    <div v-if="isHovered" class="hover-glow"></div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  card: {
    type: Object,
    required: true
  },
  index: {
    type: Number,
    required: true
  },
  polygonShape: {
    type: String,
    default: '20% 0%, 80% 0%, 100% 100%, 0% 100%' // 默认梯形
  }
})

const emit = defineEmits(['click'])

const position = ref({ x: 0, y: 0 })
const targetPosition = ref({ x: 0, y: 0 })
const size = ref({ width: 200, height: 140 })
const rotation = ref(0)
const animationId = ref(null)
const isHovered = ref(false)
const isAnimating = ref(true) // 动画状态控制

// 计算不规则尺寸
const calculateSize = () => {
  const baseWidth = 160
  const baseHeight = 110
  const widthVariation = Math.random() * 60 - 10
  const heightVariation = Math.random() * 40 - 10
  return {
    width: baseWidth + widthVariation,
    height: baseHeight + heightVariation
  }
}

// 获取初始位置 - 均匀分布
const getRandomPosition = () => {
  const containerWidth = window.innerWidth
  const containerHeight = window.innerHeight
  
  // 下半部分区域（从40%开始到85%）
  const startY = containerHeight * 0.4
  const endY = containerHeight * 0.85
  const availableHeight = endY - startY
  
  // 使用网格布局思路，但添加随机偏移
  const totalCards = 15 // 假设最多15个卡片
  const cardsPerRow = Math.ceil(Math.sqrt(totalCards))
  const row = Math.floor(props.index / cardsPerRow)
  const col = props.index % cardsPerRow
  
  const gridWidth = containerWidth / cardsPerRow
  const gridHeight = availableHeight / Math.ceil(totalCards / cardsPerRow)
  
  // 基础网格位置 + 随机偏移
  const baseX = gridWidth * col + gridWidth / 2
  const baseY = startY + gridHeight * row + gridHeight / 2
  
  const randomOffsetX = (Math.random() - 0.5) * (gridWidth * 0.4)
  const randomOffsetY = (Math.random() - 0.5) * (gridHeight * 0.4)
  
  const x = baseX + randomOffsetX - size.value.width / 2
  const y = baseY + randomOffsetY - size.value.height / 2
  
  // 边界检查
  const margin = 10
  return {
    x: Math.max(margin, Math.min(x, containerWidth - size.value.width - margin)),
    y: Math.max(startY, Math.min(y, endY - size.value.height - margin))
  }
}

// 计算新的目标位置 - 缓慢移动
const calculateNewTarget = () => {
  const containerWidth = window.innerWidth
  const containerHeight = window.innerHeight
  const startY = containerHeight * 0.4
  const endY = containerHeight * 0.85
  
  // 小范围移动，保持在当前网格附近
  const time = Date.now() / 1000
  const smallRange = 30 // 移动范围减小
  
  const waveX = Math.sin(time * 0.2 + props.index) * smallRange
  const waveY = Math.cos(time * 0.2 + props.index) * smallRange
  
  let newX = position.value.x + waveX
  let newY = position.value.y + waveY
  
  // 严格边界检查
  const maxX = containerWidth - size.value.width - 10
  const maxY = endY - size.value.height - 10
  const minY = startY
  
  newX = Math.max(10, Math.min(newX, maxX))
  newY = Math.max(minY, Math.min(newY, maxY))
  
  // 随机旋转
  if (Math.random() > 0.95) {
    rotation.value += (Math.random() - 0.5) * 2
  }
  
  return { x: newX, y: newY }
}

// 动画循环 - 简化版本
const animate = () => {
  if (!isAnimating.value) return
  
  // 检查是否悬停
  if (!isHovered.value) {
    // 检查是否到达目标位置
    const dx = targetPosition.value.x - position.value.x
    const dy = targetPosition.value.y - position.value.y
    const distance = Math.sqrt(dx * dx + dy * dy)
    
    if (distance < 2) {
      // 到达目标，计算新目标
      targetPosition.value = calculateNewTarget()
    }
    
    // 平滑移动
    const ease = 0.03
    position.value.x += dx * ease
    position.value.y += dy * ease
  }
  
  animationId.value = requestAnimationFrame(animate)
}

// 鼠标悬停处理
const handleMouseEnter = (event) => {
  console.log(`Card ${props.index} - Mouse Enter, stopping animation`)
  isHovered.value = true
  event.stopPropagation()
}

const handleMouseLeave = (event) => {
  console.log(`Card ${props.index} - Mouse Leave, resuming animation`)
  isHovered.value = false
  event.stopPropagation()
}

// 处理点击
const handleClick = () => {
  emit('click')
}

// 计算卡片样式
const cardStyle = computed(() => {
  const zIndex = isHovered.value ? 200 : 10 + props.index
  
  if (isHovered.value) {
    return {
      transform: `translate(${position.value.x}px, ${position.value.y}px) rotate(${rotation.value}deg) scale(1.08)`,
      width: `${size.value.width}px`,
      height: `${size.value.height}px`,
      clipPath: `polygon(${props.polygonShape})`,
      zIndex: zIndex,
      opacity: 1,
      pointerEvents: 'auto',
      position: 'fixed !important'
    }
  }
  
  return {
    transform: `translate(${position.value.x}px, ${position.value.y}px) rotate(${rotation.value}deg)`,
    width: `${size.value.width}px`,
    height: `${size.value.height}px`,
    clipPath: `polygon(${props.polygonShape})`,
    zIndex: zIndex,
    opacity: 1,
    pointerEvents: 'auto'
  }
})

// 响应窗口大小变化
const handleResize = () => {
  // 重新计算初始位置
  position.value = getRandomPosition()
  targetPosition.value = calculateNewTarget()
}

onMounted(() => {
  // 计算初始尺寸
  size.value = calculateSize()
  
  // 初始化位置
  position.value = getRandomPosition()
  targetPosition.value = calculateNewTarget()
  
  // 延迟启动动画
  setTimeout(() => {
    animate()
  }, 200 + props.index * 30)
  
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  isAnimating.value = false
  if (animationId.value) {
    cancelAnimationFrame(animationId.value)
  }
  window.removeEventListener('resize', handleResize)
})
</script>

<style scoped>
.floating-card {
  position: absolute;
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.95) 0%, rgba(240, 240, 240, 0.95) 100%);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.2);
  cursor: pointer;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.3);
  user-select: none;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  transition: all 0.3s ease;
}

.floating-card.is-hovered {
  position: fixed !important;
  box-shadow: 0 12px 40px rgba(102, 126, 234, 0.6), 0 0 20px rgba(102, 126, 234, 0.4);
  border-color: rgba(102, 126, 234, 0.8);
  filter: brightness(1.1);
  transform: scale(1.08) !important;
}

.floating-card:active {
  transform: scale(0.95) !important;
}

/* 发光边框效果 */
.hover-glow {
  position: absolute;
  top: -4px;
  left: -4px;
  right: -4px;
  bottom: -4px;
  background: linear-gradient(45deg, #667eea, #764ba2, #f093fb, #667eea);
  background-size: 400% 400%;
  border-radius: inherit;
  clip-path: inherit;
  opacity: 0.6;
  animation: glow-pulse 2s ease-in-out infinite;
  pointer-events: none;
  z-index: -1;
}

@keyframes glow-pulse {
  0%, 100% {
    opacity: 0.4;
    filter: blur(8px);
  }
  50% {
    opacity: 0.8;
    filter: blur(12px);
  }
}

/* 悬停内容高亮 */
.floating-card.is-hovered .card-title {
  color: #667eea;
  font-weight: 800;
  transform: scale(1.05);
}

.floating-card.is-hovered .card-url {
  color: #764ba2;
}

.floating-card.is-hovered .card-desc {
  color: #555;
}

.floating-card.is-hovered .card-category {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  transform: scale(1.1);
  box-shadow: 0 4px 12px rgba(240, 147, 251, 0.4);
}

.card-content {
  padding: 10px;
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.card-title {
  font-size: 0.9rem;
  font-weight: 700;
  color: #333;
  margin-bottom: 3px;
  line-height: 1.2;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-width: 100%;
  transition: all 0.3s ease;
}

.card-url {
  font-size: 0.65rem;
  color: #666;
  margin-bottom: 3px;
  word-break: break-all;
  font-family: 'Courier New', monospace;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-width: 100%;
}

.card-desc {
  font-size: 0.7rem;
  color: #888;
  line-height: 1.2;
  margin-bottom: 3px;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  max-width: 100%;
}

.card-category {
  display: inline-block;
  padding: 2px 6px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border-radius: 4px;
  font-size: 0.6rem;
  font-weight: 600;
  transition: all 0.3s ease;
}

/* 响应式调整 */
@media (max-width: 768px) {
  .card-content {
    padding: 6px;
  }
  
  .card-title {
    font-size: 0.75rem;
  }
  
  .card-url {
    font-size: 0.55rem;
  }
  
  .card-desc {
    font-size: 0.6rem;
  }
  
  .card-category {
    font-size: 0.5rem;
    padding: 1px 4px;
  }
}
</style>
