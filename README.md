# 音律视界 yinlv-vision

**华语数字音乐多维特征可视分析系统**：一个可交互的数据可视化大屏，从流派演化、创作者协作、声学特征和听众行为四个角度分析华语音乐。

🔗 在线演示：https://andywangaa.github.io/yinlv-vision/

## 功能

左侧的**全局控制台**负责筛选，包括时间跨度（1980 – 2025）和六种流派（流行、摇滚、民谣、R&B、嘻哈、电子）。四个视图会随筛选条件联动更新：

| 视图 | 内容 | 图表 |
|---|---|---|
| 星轨交汇 | 音乐人与幕后创作者的协作关系网络 | 力导向关系图 |
| 时代回响 | 各流派随时间的流行度变化 | 主题河流图 |
| 爆款基因 | 舞蹈性、能量感、声学度等声学特征的分布 | 雷达图 + 平行坐标 |
| 听众画像 | 收听时段分布与用户转化路径 | 热力图 + 桑基图 |

## 技术栈

- [Vue 3](https://vuejs.org/)（`<script setup>`）
- [ECharts 6](https://echarts.apache.org/)
- [Vite](https://vitejs.dev/)
- 通过 `gh-pages` 部署到 GitHub Pages

## 本地运行

需要先安装 [Node.js](https://nodejs.org/)。

```bash
npm install     # 安装依赖
npm run dev     # 启动开发服务器
npm run build   # 打包到 dist/
```

## 数据说明

数据存放在 `public/data/` 目录下的 JSON 文件中。
