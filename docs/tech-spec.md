# 🔧 旅行足迹 - 技术规范

> 版本：v1.0 | 最后更新：2026-06-24

---

## 一、技术栈

| 层级 | 技术 | 说明 |
|------|------|------|
| 3D 渲染 | **Three.js** (r160+) | WebGL 3D 引擎 |
| 地球库 | **Globe.GL** | 基于 Three.js 的地球组件，支持国家轮廓、光点、交互 |
| 国家数据 | **GeoJSON** (Natural Earth 110m) | 国家边界数据 |
| 样式 | **原生 CSS** | 星空背景用 CSS animation + 渐变 |
| 逻辑 | **原生 JavaScript** (ES6+) | 无框架，保持轻量 |
| 存储 | **localStorage** | 浏览器本地存储 |
| 依赖加载 | **CDN (unpkg/jsdelivr)** | 无需 npm，直接从 CDN 加载 |

---

## 二、项目文件结构

```
D:/旅行足迹/
├── index.html              # 主页面入口
├── css/
│   └── style.css           # 全局样式（星空背景、面板、按钮等）
├── js/
│   ├── app.js              # 应用主入口，初始化各模块
│   ├── globe.js            # 3D 地球初始化、渲染、国家高亮逻辑
│   ├── search.js           # 搜索功能：模糊搜索国家/城市
│   ├── stats.js            # 统计面板：国家数、城市数、面积、最远距离
│   ├── data.js             # 数据管理：localStorage 读写、导入导出
│   ├── ui.js               # UI 交互：面板切换、年份筛选、提示消息
│   └── country-info.js     # 国家信息侧边栏
├── data/
│   ├── countries.json      # 国家基本信息（名称、面积、首都、简介等）
│   └── geodata/            # GeoJSON 国家边界数据（按需加载）
├── docs/
│   ├── requirements.md     # 需求文档
│   ├── tech-spec.md        # 本文件 - 技术规范
│   ├── design-spec.md      # 设计规范
│   └── execution-plan.md   # 执行步骤
├── assets/
│   └── textures/           # 地球贴图（可选）
├── CLAUDE.md               # AI 助手指引文件
└── README.md               # 项目说明
```

---

## 三、核心依赖清单

| 库 | CDN 地址 | 用途 |
|----|----------|------|
| Three.js | `https://unpkg.com/three@0.160.0/build/three.min.js` | 3D 引擎 |
| Globe.GL | `https://unpkg.com/globe.gl@2.32.0/dist/globe.gl.min.js` | 3D 地球 |
| OrbitControls (可选) | Three.js addon | 鼠标控制 |

---

## 四、数据模型

### 4.1 用户数据（localStorage 存储）

```javascript
{
  homeCity: { name: "北京", lat: 39.9, lng: 116.4 },   // 常驻城市
  visitedCountries: [                                      // 去过的国家列表
    {
      code: "CN",           // ISO 3166-1 alpha-2 国家代码
      name: "中国",
      visitDates: ["2023-05-10", "2024-08-15"],           // 多次访问记录
      visitedCities: [                                     // 在该国去过的城市
        { name: "上海", lat: 31.2, lng: 121.5, visitDates: ["2023-05-10"] },
        { name: "北京", lat: 39.9, lng: 116.4, visitDates: ["2024-08-15"] }
      ]
    }
  ],
  version: 1                // 数据版本号，用于未来升级
}
```

### 4.2 国家信息数据（静态 JSON）

```javascript
{
  "CN": {
    name: "中国",
    capital: "北京",
    area: 9634057,          // 平方公里
    population: "约 14 亿",
    continent: "亚洲",
    languages: ["中文"],
    currency: "人民币 (CNY)",
    intro: "中国是世界四大文明古国之一..."  // 简短介绍
  }
}
```

---

## 五、关键实现方案

### 5.1 3D 地球渲染
- 使用 Globe.GL，传入 GeoJSON 数据渲染国家边界
- 通过 `hexColor()` 回调控制每个国家的颜色（去过=亮色，未去过=暗色）
- 使用 Three.js Raycaster 实现点击选中

### 5.2 全球 → 国家视角切换
- 点击国家时，使用 Globe.GL 的 `pointOfView()` 方法平滑过渡到该国经纬度
- 调整相机距离（zoom in）
- 在城市级别显示光点（scatter plot dots）

### 5.3 搜索功能
- 合并国家列表 + 城市列表作为搜索源
- 使用简单的字符串模糊匹配（`String.includes()`）
- 搜索结果按匹配度排序

### 5.4 面积计算
- 直接读取 countries.json 中预存的面积数据
- 合计所有去过国家的面积

### 5.5 最远距离计算
- 使用 Haversine 公式计算球面距离
- 以常驻城市坐标为起点，遍历所有去过的城市，取最大值

### 5.6 年份筛选
- 遍历 visitedCountries → visitedCities，按 visitDates 中的年份过滤
- 根据筛选结果动态更新地球高亮状态

---

## 六、浏览器兼容性

| 浏览器 | 最低版本 |
|--------|----------|
| Chrome | 90+ |
| Edge | 90+ |
| Firefox | 90+ |

> 需要浏览器支持 WebGL 和 ES6 模块 / ES6 语法。

---

## 七、部署方案

| 平台 | 方式 | 费用 |
|------|------|------|
| **GitHub Pages**（推荐） | 推送到 GitHub 仓库，自动部署 | 免费 |
| Netlify | 拖拽文件夹上传 | 免费 |
| Vercel | 关联 Git 仓库 | 免费 |

最终公网地址示例：`https://你的用户名.github.io/travel-footprint/`
