# Vue3 智能导航系统

一个纯前端的Vue3导航项目，采用**上下分区布局**：上半部分是搜索和分类网格，下半部分是随机移动的不规则多边形卡片，完美解决闪烁问题。

## 功能特点

- 🎨 **智能上下布局**: 
  - 上半部分：搜索框 + 分类网格（点击筛选）
  - 下半部分：随机移动的不规则多边形卡片
- 🔍 **实时搜索**: 支持搜索网站名称、描述、分类和URL
- 🎯 **多边形卡片**: 
  - 使用 `clip-path` 实现梯形、五边形、菱形等不规则形状
  - 每个卡片都有不同的尺寸和随机旋转
  - 仅在下半部分区域移动，永不越界
- ✨ **无闪烁设计**: 
  - 完全移除悬停暂停功能
  - 卡片持续平滑移动，无任何中断
  - 纯CSS过渡，无JavaScript状态冲突
- 🔗 **网站导航**: 从 `public/link.json` 读取链接数据
- 📱 **响应式设计**: 适配不同屏幕尺寸
- ⚡ **纯前端**: 无需后端，可部署在Vercel等静态托管平台

## 项目结构

```
├── public/
│   └── link.json          # 网站链接数据
├── src/
│   ├── components/
│   │   └── FloatingCard.vue  # 浮动卡片组件
│   ├── App.vue            # 主应用组件
│   └── main.js            # 应用入口
├── index.html             # HTML入口
├── vite.config.js         # Vite配置
├── package.json           # 项目依赖
├── vercel.json           # Vercel部署配置
└── README.md             # 项目说明
```

## 数据格式

`link.json` 文件格式要求：

```json
[
  {
    "name": "网站名称",
    "url": "https://example.com",
    "catelog": "分类",
    "desc": "网站描述"
  }
]
```

## 安装和运行

### 本地开发

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build

# 预览生产版本
npm run preview
```

### 部署到Vercel

1. 将项目推送到GitHub仓库
2. 登录Vercel并导入GitHub仓库
3. Vercel会自动检测并部署

## 使用说明

1. 首次运行时，项目会自动从 `public/link.json` 读取数据
2. 如果 `public/link.json` 不存在，会使用内置的示例数据
3. 卡片会在屏幕上随机移动，鼠标悬停时会暂停
4. 点击任意卡片即可在新标签页中打开对应网站
5. 可以修改 `link.json` 文件来添加或更新链接

## 自定义配置

### 修改卡片数量

在 `src/App.vue` 中修改 `maxCards` 变量：

```javascript
const maxCards = ref(20) // 默认显示15个卡片
```

### 修改动画速度

在 `src/components/FloatingCard.vue` 中修改动画参数：

```javascript
const ease = 0.02 // 移动平滑度
const time = Date.now() / 1000 // 动画时间因子
```

## 技术栈

- Vue 3.3
- Vite 4.4
- Element Plus (用于加载状态)
- 纯JavaScript动画 (requestAnimationFrame)

## 浏览器兼容性

- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

## 许可证

MIT License