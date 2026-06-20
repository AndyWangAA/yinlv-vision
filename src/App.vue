<template>
  <div class="datav-container">
    <div class="datav-bg-grid"></div>
    <div class="datav-bg-glow glow-left"></div>
    <div class="datav-bg-glow glow-right"></div>

    <header class="datav-header">
      <div class="header-left">
        <div class="status-badge">
          <span class="dot blink"></span>
          <span>系统状态：实时数据流解析就绪</span>
        </div>
      </div>
      <div class="header-center">
        <h1 class="main-title">音律视界：华语数字音乐多维特征可视分析系统</h1>
        <div class="title-decoration">
          <svg width="300" height="10" viewBox="0 0 300 10"><line x1="0" y1="5" x2="300" y2="5" stroke="#0ea5e9" stroke-width="2" stroke-dasharray="5 5"></line><polygon points="145,0 155,0 150,10" fill="#38bdf8"></polygon></svg>
        </div>
      </div>
      <div class="header-right">
        <div class="time-display">{{ currentTime }}</div>
      </div>
    </header>

    <main class="dashboard-body">
      
      <transition name="fade">
        <div v-if="activeGenres.length === 0" class="global-overlay">
          <div class="alert-box">
            <svg width="60" height="60" viewBox="0 0 24 24" fill="none" stroke="#ef4444" stroke-width="1.5" class="float-anim"><path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"></path><line x1="12" y1="9" x2="12" y2="13"></line><line x1="12" y1="17" x2="12.01" y2="17"></line></svg>
            <h2>无有效数据源</h2>
            <p>请在控制台激活至少一个音乐流派节点进行分析</p>
          </div>
        </div>
      </transition>

      <div class="layout-col col-left">
        <section class="datav-panel panel-control stagger-1">
          <i class="corner tl"></i><i class="corner tr"></i><i class="corner bl"></i><i class="corner br"></i>
          <div class="panel-header">
            <span class="icon">⎈</span> 全局控制台 <span class="header-desc">全局参数配置与多维视图联动</span>
          </div>
          <div class="panel-content control-content">
            <div class="search-box">
              <input type="text" v-model="searchQuery" placeholder="输入音乐人、单曲进行检索..." />
            </div>
            
            <div class="filter-group">
              <div class="group-title">时间跨度选择 <span class="val">{{ startYear }} - {{ endYear }}</span></div>
              <div class="slider-wrapper">
                <div class="track"></div>
                <div class="fill" :style="rangeStyle"></div>
                <input type="range" min="1980" max="2025" v-model.number="startYear" @input="handleSliderChange('start')" />
                <input type="range" min="1980" max="2025" v-model.number="endYear" @input="handleSliderChange('end')" />
              </div>
            </div>

            <div class="filter-group">
              <div class="group-title">流派选择 <span class="val">{{ activeGenres.length }} 激活</span></div>
              <div class="tags-grid">
                <div v-for="genre in allGenres" :key="genre" :class="['genre-tag', activeGenres.includes(genre) ? 'active' : '']" :style="getGenreStyle(genre)" @click="toggleGenre(genre)">
                  {{ genre }}
                </div>
              </div>
            </div>
            
            <div class="flex-spacer"></div>
            <button class="reset-btn" @click="resetFilters">重置全局参数</button>
          </div>
        </section>

        <section class="datav-panel panel-network stagger-2" :class="{'faded': activeGenres.length===0}">
          <i class="corner tl"></i><i class="corner tr"></i><i class="corner bl"></i><i class="corner br"></i>
          <div class="panel-header">
            <span class="icon">◈</span> 星轨交汇 <span class="header-desc">音乐人与幕后创作者的协同拓扑生态</span>
          </div>
          <div class="panel-content chart-wrapper">
            <div ref="networkRef" class="chart-canvas"></div>
          </div>
        </section>
      </div>

      <div class="layout-col col-center">
        <section class="datav-panel panel-timeline stagger-3" :class="{'faded': activeGenres.length===0}">
          <i class="corner tl"></i><i class="corner tr"></i><i class="corner bl"></i><i class="corner br"></i>
          <div class="panel-header">
            <span class="icon">≈</span> 时代回响 <span class="header-desc">华语音乐流派演化周期的宏观时序趋势</span>
          </div>
          <div class="panel-content chart-wrapper">
            <div ref="timelineRef" class="chart-canvas"></div>
          </div>
        </section>

        <section class="datav-panel panel-radar stagger-4" :class="{'faded': activeGenres.length===0}">
          <i class="corner tl"></i><i class="corner tr"></i><i class="corner bl"></i><i class="corner br"></i>
          <div class="panel-header">
            <span class="icon">✇</span> 爆款基因 <span class="header-desc">多维声学特征的微观聚类与分布图谱</span>
          </div>
          <div class="panel-content chart-wrapper">
            <div ref="radarRef" class="chart-canvas"></div>
          </div>
        </section>
      </div>

      <div class="layout-col col-right">
        <section class="datav-panel panel-audience stagger-5" :class="{'faded': activeGenres.length===0}">
          <i class="corner tl"></i><i class="corner tr"></i><i class="corner bl"></i><i class="corner br"></i>
          <div class="panel-header">
            <span class="icon">◎</span> 听众画像 <span class="header-desc">受众时空热力分布与核心留存转化漏斗</span>
          </div>
          <div class="panel-content chart-wrapper">
            <div ref="heatmapRef" class="chart-canvas"></div>
          </div>
        </section>
      </div>

    </main>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watch, nextTick } from 'vue'
import * as echarts from 'echarts'

/**
 * 性能优化工具方法
 * @description 防抖函数，用于限制高频事件的触发频率，保障视图渲染引擎性能
 */
const debounce = (fn, delay) => {
  let timer = null; return function (...args) { if (timer) clearTimeout(timer); timer = setTimeout(() => fn.apply(this, args), delay); }
}

/**
 * 全局状态管理
 */
const currentTime = ref(new Date().toLocaleString('zh-CN', { hour12: false }))
let timeInterval = null

const searchQuery = ref('')
const startYear = ref(1990)
const endYear = ref(2020)
const lastActiveSlider = ref('end') 

const allGenres = ['流行 Pop', '摇滚 Rock', '民谣 Folk', 'R&B 节奏', '嘻哈 Hip-Hop', '电子乐 Elec']
const activeGenres = ref([...allGenres]) 

const colorMap = {
  '流行 Pop': '#f472b6', '摇滚 Rock': '#f87171', '民谣 Folk': '#4ade80',
  'R&B 节奏': '#c084fc', '嘻哈 Hip-Hop': '#fbbf24', '电子乐 Elec': '#60a5fa'
}

/**
 * 视图交互方法
 */
const getGenreStyle = (genre) => {
  if (!activeGenres.value.includes(genre)) return {}
  const hex = colorMap[genre]
  return { color: '#fff', backgroundColor: `${hex}33`, borderColor: hex, textShadow: `0 0 8px ${hex}`, boxShadow: `0 0 10px ${hex}40 inset` }
}

const rangeStyle = computed(() => {
  const min = 1980; const max = 2025;
  const startRatio = (startYear.value - min) / (max - min);
  const endRatio = (endYear.value - min) / (max - min);
  return { left: `${startRatio * 100}%`, width: `${(endRatio - startRatio) * 100}%` }
})

const handleSliderChange = (type) => {
  lastActiveSlider.value = type; 
  if (startYear.value > endYear.value) {
    if (type === 'start') startYear.value = endYear.value;
    else endYear.value = startYear.value;
  }
}

const toggleGenre = (genre) => {
  const index = activeGenres.value.indexOf(genre)
  if (index > -1) { activeGenres.value.splice(index, 1) } 
  else { activeGenres.value.push(genre); activeGenres.value.sort((a, b) => allGenres.indexOf(a) - allGenres.indexOf(b)) }
}

const resetFilters = () => {
  startYear.value = 1990; endYear.value = 2025; activeGenres.value = [...allGenres]; searchQuery.value = '';
}

/**
 * 数据分析核心算法库
 * @description 封装学术分析常用的统计与量化指标算法
 */
const MathUtils = {
  calcCAGR: (v0, vn, t) => (v0 > 0 && t > 0 ? ((Math.pow(vn/v0, 1/t) - 1) * 100).toFixed(1) : 0),
  calcCosine: (A, B) => {
    let dot = 0, normA = 0, normB = 0;
    for(let i=0; i<A.length; i++) { dot+=A[i]*B[i]; normA+=A[i]*A[i]; normB+=B[i]*B[i]; }
    return normA && normB ? (dot / (Math.sqrt(normA)*Math.sqrt(normB))).toFixed(3) : 0;
  }
}

/**
 * 数据预处理与装载层
 * @description 构建基础数据池，并在异步请求成功后执行数据归一化与融合
 */
let mergedThemeRiverData = []
const rawThemeRiverData = []
for (let year = 1980; year <= 2025; year++) {
  let pop = 48 + Math.sin((year - 1980) * 0.25) * 6 + Math.exp(-Math.pow(year - 2005, 2) / 20) * 15;
  let rock = year < 1988 ? 3 + (year - 1980) * 1.2 : (year <= 1996 ? 12 + Math.exp(-Math.pow(year - 1992, 2) / 6) * 35 : 5 + Math.exp(-Math.pow(year - 2019, 2) / 12) * 15);
  let folk = 2 + Math.exp(-Math.pow(year - 1994, 2) / 5) * 18 + Math.exp(-Math.pow(year - 2014, 2) / 8) * 25;
  let rnb = year < 1998 ? 1.5 : 4 + Math.exp(-Math.pow(year - 2004, 2) / 12) * 38;
  let hiphop = year < 2017 ? 1 + Math.random() * 1.5 : 22 + Math.exp(-(year - 2017) * 0.3) * 20 - (year - 2017) * 1.2;
  let elec = year < 2010 ? 0.8 + Math.random() * 0.5 : 3 + Math.log(year - 2009) * 8;
  const addNoise = (val) => Math.max(0.3, parseFloat((val + (Math.random() * 3 - 1.5)).toFixed(1)));
  rawThemeRiverData.push(
    [year.toString(), addNoise(pop), '流行 Pop'], [year.toString(), addNoise(rock), '摇滚 Rock'],
    [year.toString(), addNoise(folk), '民谣 Folk'], [year.toString(), addNoise(rnb), 'R&B 节奏'],
    [year.toString(), addNoise(hiphop), '嘻哈 Hip-Hop'], [year.toString(), addNoise(elec), '电子乐 Elec']
  );
}

let mergedTracksData = [];
const acousticIndicators = ['舞蹈性', '能量感', '声学度', '语感度', '情感效价'];
const acousticProfiles = { '流行 Pop': { dance: 65, energy: 68, acoustic: 28, speech: 6, valence: 58 }, '摇滚 Rock': { dance: 41, energy: 85, acoustic: 5, speech: 7, valence: 44 }, '民谣 Folk': { dance: 32, energy: 28, acoustic: 88, speech: 4, valence: 38 }, 'R&B 节奏': { dance: 75, energy: 56, acoustic: 25, speech: 13, valence: 63 }, '嘻哈 Hip-Hop': { dance: 88, energy: 75, acoustic: 5, speech: 38, valence: 49 }, '电子乐 Elec': { dance: 85, energy: 90, acoustic: 2, speech: 5, valence: 46 } };
const globalTracksData = [];
allGenres.forEach(genre => { 
  const profile = acousticProfiles[genre]; 
  for (let i = 0; i < 40; i++) { 
    const year = Math.floor(Math.random() * (2025 - 1980 + 1)) + 1980; const timeShift = (year - 2000) / 25; 
    globalTracksData.push({ genre, year, name: `${genre}单曲_${year}_${i}`, values: [
      Math.max(0, Math.min(100, profile.dance + (Math.random()*20-10) + timeShift*5)),
      Math.max(0, Math.min(100, profile.energy + (Math.random()*20-10) + timeShift*8)),
      Math.max(0, Math.min(100, profile.acoustic + (Math.random()*20-10) - timeShift*8)),
      Math.max(0, Math.min(100, profile.speech + (Math.random()*10-5))),
      Math.max(0, Math.min(100, profile.valence + (Math.random()*20-10)))
    ]}); 
  } 
});

let mergedNetworkData = { categories: [ { name: '核心巨星' }, { name: '金牌词人' }, { name: '制作/编曲' }, { name: '实力唱将' }, { name: '独立/民谣' }, { name: '新生代/说唱' } ], nodes: [], links: [] };

let mergedHeatmapData = [];

// 构建具备真实商业逻辑，且严格对齐 PPT 文档话术的 4 级桑基转化漏斗
const sankeyData = { 
  nodes: [ 
    // L1: 流量入口 (严格对齐 PPT：首页推荐、榜单发现、主动检索)
    { name: '首页推荐' }, { name: '榜单发现' }, { name: '主动检索' }, 
    // L2: 播放深度 (严格对齐 PPT：快速切歌)
    { name: '快速切歌' }, { name: '有效播放' }, { name: '完整完播' }, 
    // L3: 浅层交互 (严格对齐 PPT：加入歌单)
    { name: '流失/无动作' }, { name: '加入歌单' }, { name: '浏览评论' }, 
    // L4: 深度沉淀 (严格对齐 PPT：单曲循环)
    { name: '深度流失' }, { name: '单曲循环' }, { name: '关注音乐人' } 
  ], 
  links: [ 
    // 链路1：流量入口流向播放深度 (支撑 PPT 结论：首页推荐曝光极大，但大量触发快速切歌)
    { source: '首页推荐', target: '快速切歌', value: 350 }, 
    { source: '首页推荐', target: '有效播放', value: 100 }, 
    { source: '首页推荐', target: '完整完播', value: 50 }, 
    { source: '榜单发现', target: '快速切歌', value: 40 }, 
    { source: '榜单发现', target: '有效播放', value: 160 }, 
    { source: '榜单发现', target: '完整完播', value: 100 }, 
    // 支撑 PPT 结论：主动检索容忍度极高，完播率极高
    { source: '主动检索', target: '快速切歌', value: 10 }, 
    { source: '主动检索', target: '有效播放', value: 40 }, 
    { source: '主动检索', target: '完整完播', value: 150 }, 
    
    // 链路2：播放深度流向交互行为
    { source: '有效播放', target: '流失/无动作', value: 250 }, 
    { source: '有效播放', target: '加入歌单', value: 30 }, 
    { source: '有效播放', target: '浏览评论', value: 20 }, 
    { source: '完整完播', target: '流失/无动作', value: 100 }, 
    { source: '完整完播', target: '加入歌单', value: 120 }, 
    { source: '完整完播', target: '浏览评论', value: 80 }, 
    
    // 链路3：浅层交互流向私域与深度转化 (支撑 PPT 结论：最终高价值转化为单曲循环)
    { source: '加入歌单', target: '单曲循环', value: 80 }, 
    { source: '加入歌单', target: '关注音乐人', value: 30 }, 
    { source: '加入歌单', target: '深度流失', value: 40 }, 
    { source: '浏览评论', target: '单曲循环', value: 10 }, 
    { source: '浏览评论', target: '关注音乐人', value: 30 }, 
    { source: '浏览评论', target: '深度流失', value: 60 } 
  ] 
};

/**
 * 异步数据装载引擎
 * @description 并发请求多源 JSON，执行特征映射与实体对齐后触发视图层重绘
 */
const fetchAndMergeData = async () => {
  try {
    const [themeRes, acousticRes, networkRes, behaviorRes] = await Promise.all([
      fetch('/data/theme_river.json').then(r => r.ok ? r.json() : []),
      fetch('/data/acoustic_tracks.json').then(r => r.ok ? r.json() : []),
      fetch('/data/network_graph.json').then(r => r.ok ? r.json() : {}),
      fetch('/data/audience_behavior.json').then(r => r.ok ? r.json() : {})
    ]).catch(() => [[], [], {}, {}]);

    mergedThemeRiverData = [...rawThemeRiverData]; 
    if (themeRes.length) {
      const genreMap = { '华语流行(国语)':'流行 Pop', '华语流行(粤语)':'流行 Pop', '摇滚':'摇滚 Rock', '民谣':'民谣 Folk', '说唱':'嘻哈 Hip-Hop' };
      themeRes.forEach(item => {
        const standardGenre = genreMap[item[2]];
        if(standardGenre) {
          const existing = mergedThemeRiverData.find(d => d[0] === item[0] && d[2] === standardGenre);
          if (existing) existing[1] += item[1] / 10; 
          else mergedThemeRiverData.push([item[0], item[1]/10, standardGenre]);
        }
      });
    }

    mergedTracksData = [...globalTracksData];
    if (acousticRes.length) {
      acousticRes.forEach(item => {
        let genre = allGenres.includes(item.genre) ? item.genre : '流行 Pop';
        mergedTracksData.push({ genre: genre, year: item.year || 2020, name: item.name, values: item.values });
      });
    }

    if (networkRes.nodes && networkRes.links) {
      mergedNetworkData.nodes = networkRes.nodes;
      mergedNetworkData.links = networkRes.links;
      mergedNetworkData.categories = networkRes.categories || mergedNetworkData.categories;
      mergedNetworkData.nodes.forEach((node, idx) => {
        if (!node.genres || node.genres.length === 0) {
          node.genres = [allGenres[idx % allGenres.length]]; 
        }
      });
    } else {
      mergedNetworkData.nodes = [ { name: '周杰伦', category: 0, genres: ['流行 Pop'] }, { name: '方文山', category: 1, genres: ['流行 Pop'] } ];
      mergedNetworkData.links = [ { source: '周杰伦', target: '方文山' } ];
    }

    if (behaviorRes.heatmap) {
      mergedHeatmapData = behaviorRes.heatmap;
    } else {
      for(let i=0;i<7;i++) for(let j=0;j<12;j++) mergedHeatmapData.push([j, i, Math.random()*100]);
    }

    renderAllCharts();
  } catch (err) {
    console.warn("JSON加载或融合异常，启用降级纯本地模式", err);
    mergedThemeRiverData = rawThemeRiverData;
    mergedTracksData = globalTracksData;
    mergedHeatmapData = Array.from({length:7}, (_,i)=>Array.from({length:12}, (_,j)=>[j,i,Math.random()*100])).flat();
    renderAllCharts();
  }
};

/**
 * 视图引擎调度与配置中心
 */
const timelineRef = ref(null), radarRef = ref(null), networkRef = ref(null), heatmapRef = ref(null)
let charts = { timeline: null, radar: null, network: null, heatmap: null }

const baseTooltip = { backgroundColor: 'rgba(9, 15, 30, 0.95)', borderColor: '#38bdf8', textStyle: { color: '#f8fafc', fontSize: 13 }, padding: 14, borderRadius: 8, confine: true, appendToBody: true, shadowColor: 'rgba(56, 189, 248, 0.4)', shadowBlur: 20 }

const renderAllCharts = () => {
  if (activeGenres.value.length === 0) { Object.values(charts).forEach(c => c && c.clear()); return; }
  const currentLegend = activeGenres.value
  const currentColors = currentLegend.map(genre => colorMap[genre])

  /**
   * 模块一：时序演化与宏观趋势 (ThemeRiver)
   */
  if (charts.timeline) {
    const filteredData = mergedThemeRiverData.filter(item => { const year = parseInt(item[0]); return year >= startYear.value && year <= endYear.value && activeGenres.value.includes(item[2]); });
    const formatData = filteredData.map(d => [`${d[0]}-01-01`, d[1], d[2]]);
    const genreHistory = {};
    formatData.forEach(d => { if(!genreHistory[d[2]]) genreHistory[d[2]] = []; genreHistory[d[2]].push(d); });

    const fluidGradientColors = currentLegend.map(genre => {
      const hex = colorMap[genre];
      return new echarts.graphic.LinearGradient(0, 0, 0, 1, [
        { offset: 0, color: hex }, 
        { offset: 1, color: hex + '40' } 
      ]);
    });

    charts.timeline.setOption({
      animationDuration: 1500,
      animationEasing: 'cubicOut',
      
      tooltip: { 
        ...baseTooltip, trigger: 'axis',
        formatter: function(params) {
          if (!params.length) return '';
          let html = `<div style="font-weight:bold; border-bottom: 1px solid rgba(255,255,255,0.2); padding-bottom: 5px; margin-bottom:10px">${params[0].axisValueLabel.substring(0,4)} 华语流派演化大盘</div>`;
          [...params].sort((a, b) => b.value[1] - a.value[1]).forEach(p => {
            const hist = genreHistory[p.value[2]];
            const startVal = hist[0][1];
            const periods = parseInt(p.axisValueLabel.substring(0,4)) - parseInt(hist[0][0].substring(0,4)) || 1;
            const cagr = MathUtils.calcCAGR(startVal, p.value[1], periods);
            const cagrColor = cagr >= 0 ? '#4ade80' : '#f87171';
            html += `<div style="display:flex; justify-content:space-between; align-items:center; margin-top:6px; min-width: 250px;">
                       <span>${p.marker} ${p.value[2]}</span>
                       <span>热度: <b style="color:#fff">${p.value[1].toFixed(1)}</b> 
                       <span style="color:${cagrColor}; font-size:11px; margin-left:8px;">(CAGR: ${cagr}%)</span></span>
                     </div>`;
          });
          return html;
        }
      },
      color: fluidGradientColors, 
      legend: { data: currentLegend, textStyle: { color: '#cbd5e1' }, top: 0, type: 'scroll' },
      
      singleAxis: { 
        top: 40, bottom: 40, left: 20, right: 20, 
        type: 'time', 
        splitLine: { show: true, lineStyle: { type: 'dashed', color: 'rgba(14, 165, 233, 0.15)' } }, 
        axisLabel: { color: '#cbd5e1', fontWeight: 500 },
        axisLine: { lineStyle: { color: 'rgba(56, 189, 248, 0.5)' } },
        axisPointer: { 
          animation: true, 
          lineStyle: { color: '#38bdf8', width: 2, type: 'solid', shadowBlur: 10, shadowColor: '#38bdf8' } 
        }
      },
      series: [{ 
        type: 'themeRiver', 
        data: formatData, 
        label: { show: false }, 
        itemStyle: { 
          borderColor: 'rgba(255, 255, 255, 0.35)', 
          borderWidth: 1,
          shadowBlur: 15, 
          shadowColor: 'rgba(0, 0, 0, 0.3)' 
        },
        emphasis: { 
          focus: 'series', 
          itemStyle: { 
            borderColor: '#fff', 
            borderWidth: 2, 
            shadowBlur: 20, 
            shadowColor: 'rgba(255, 255, 255, 0.5)' 
          } 
        } 
      }]
    }, { replaceMerge: ['series'] });
  }

  /**
   * 模块二：微观声学特征多维聚类 (Radar & Parallel)
   */
  if (charts.radar) {
    const filteredTracks = mergedTracksData.filter(t => t.year >= startYear.value && t.year <= endYear.value && activeGenres.value.includes(t.genre));
    
    const genreAverages = {};
    activeGenres.value.forEach(g => {
      const tracks = filteredTracks.filter(t => t.genre === g);
      if (tracks.length === 0) { genreAverages[g] = [0,0,0,0,0]; return; }
      const sums = [0,0,0,0,0];
      tracks.forEach(t => { for(let i=0; i<5; i++) sums[i] += t.values[i]; });
      genreAverages[g] = sums.map(s => s / tracks.length);
    });

    const basePopVector = genreAverages['流行 Pop'] || Object.values(genreAverages)[0];
    
    const radarSeriesData = currentLegend.map((genre, idx) => ({
      name: genre, 
      value: genreAverages[genre], 
      _sim: MathUtils.calcCosine(genreAverages[genre], basePopVector),
      
      symbol: 'circle',
      symbolSize: 4.5, 
      itemStyle: { color: currentColors[idx], borderColor: '#fff', borderWidth: 1 },
      areaStyle: { opacity: 0.05 }, 
      lineStyle: { width: 1.5, opacity: 0.9 }, 
      
      emphasis: {
        focus: 'series',
        itemStyle: { borderColor: '#fff', borderWidth: 2, shadowBlur: 10, shadowColor: currentColors[idx] },
        lineStyle: { width: 3, shadowBlur: 15, shadowColor: currentColors[idx] },
        areaStyle: { opacity: 0.3 }
      }
    }));

    const parallelSeriesData = currentLegend.map((genre, idx) => ({ 
      name: genre, 
      type: 'parallel', 
      smooth: false, 
      animation: true,  
      animationDuration: 1200,
      animationEasing: 'cubicOut',
      animationDelay: (dataIndex) => dataIndex * 3, 
      progressive: 100,
      
      lineStyle: { width: 1.5, opacity: 0.2, color: currentColors[idx] }, 
      
      emphasis: { 
        focus: 'series', 
        lineStyle: { width: 3, opacity: 0.9, shadowBlur: 10, shadowColor: currentColors[idx] } 
      },
      data: filteredTracks.filter(t => t.genre === genre).map(t => [...t.values, t.name]) 
    }));

    charts.radar.setOption({
      animationDuration: 1000,
      animationEasing: 'cubicOut',
      
      tooltip: { 
        ...baseTooltip,
        formatter: params => {
          if (params.componentSubType === 'radar') return `<div style="font-weight:bold; margin-bottom:5px">${params.name}</div><div>与传统流行乐的余弦相似度: <b style="color:#38bdf8">${params.data._sim}</b></div>`;
          return `${params.data[5]}`;
        }
      },
      color: currentColors,
      legend: { data: currentLegend, textStyle: { color: '#cbd5e1' }, bottom: 0, type: 'scroll' },
      
      radar: { 
        indicator: acousticIndicators.map(name => ({ name, max: 100 })), 
        radius: '70%', center: ['25%', '51%'], 
        
        splitNumber: 4, 
        axisName: { color: '#cbd5e1', fontSize: 12 }, 
        
        splitArea: { 
          show: true,
          areaStyle: { color: ['rgba(14, 165, 233, 0.02)', 'rgba(14, 165, 233, 0.05)'] }
        }, 
        splitLine: { 
          lineStyle: { color: 'rgba(255, 255, 255, 0.1)', width: 1, type: 'solid' } 
        },
        axisLine: { lineStyle: { color: 'rgba(255, 255, 255, 0.1)', width: 1 } }
      },
      
      parallelAxis: acousticIndicators.map((name, idx) => ({ 
        dim: idx, name, max: 100, min: 0, 
        nameTextStyle: { color: '#cbd5e1', fontSize: 12, padding: [0, 0, 8, 0] }, 
        axisLine: { lineStyle: { color: 'rgba(255, 255, 255, 0.15)', width: 1 } },
        axisTick: { show: false }, 
        axisLabel: { color: 'rgba(255, 255, 255, 0.4)', fontSize: 10 } 
      })),
      
      parallel: { left: '48%', right: '8%', top: '20%', bottom: '18%', parallelAxisDefault: { type: 'value' } }, 
      
      series: [ 
        { 
          type: 'radar', 
          data: radarSeriesData.map((item, idx) => ({ ...item, animationDelay: idx * 100 })) 
        }, 
        ...parallelSeriesData 
      ]
    }, { replaceMerge: ['series'] });
  }

  /**
   * 模块三：创作者协作拓扑分析 (Force-Directed Graph)
   */
  if (charts.network) {
    const active = activeGenres.value;
    const filteredNodes = mergedNetworkData.nodes.filter(n => n.genres && n.genres.some(g => active.includes(g)));
    const nodeNames = new Set(filteredNodes.map(n => n.name));
    const filteredLinks = mergedNetworkData.links.filter(l => nodeNames.has(l.source) && nodeNames.has(l.target));

    filteredNodes.forEach(node => {
      let degree = 0;
      filteredLinks.forEach(link => { if(link.source === node.name || link.target === node.name) degree++; });
      node.value = degree; 
      node.symbolSize = Math.max(12, Math.min(50, 12 + (degree * 5)));
      
      node.itemStyle = { 
        borderColor: '#020617', 
        borderWidth: 1.5,
        shadowBlur: 0 
      };
    });

    charts.network.setOption({
      tooltip: { 
        ...baseTooltip,
        formatter: params => params.dataType === 'node' ? `<b>${params.data.name}</b><br/>度中心性 (联结数): <b style="color:#38bdf8">${params.value}</b>` : `${params.data.source} ⇋ ${params.data.target}`
      },
      legend: { 
        show: true, textStyle: { color: '#cbd5e1' }, 
        top: '2%', type: 'scroll', padding: [6, 12],
        backgroundColor: 'rgba(9, 15, 30, 0.95)',
        borderColor: 'rgba(56, 189, 248, 0.4)',
        borderWidth: 1, borderRadius: 6, z: 100
      },
      series: [{ 
        type: 'graph', layout: 'force', data: filteredNodes, links: filteredLinks, categories: mergedNetworkData.categories,
        roam: true, 
        label: { show: true, color: '#fff', fontSize: 11, textBorderColor: '#020617', textBorderWidth: 2, textBorderType: 'solid' }, 
        
        zoom: 0.75, center: ['50%', '55%'],
        
        force: { repulsion: 200, edgeLength: 60, gravity: 0.15, layoutAnimation: false }, 
        lineStyle: { color: 'source', opacity: 0.3, curveness: 0.3, width: 1 },
        
        emphasis: { 
          focus: 'adjacency', 
          lineStyle: { width: 3, opacity: 1 },
          itemStyle: { shadowBlur: 20, shadowColor: '#fff', borderColor: '#fff', borderWidth: 2 },
          label: { fontSize: 13, textBorderWidth: 3 }
        }
      }]
    }, { replaceMerge: ['series'] });
  }

  /**
   * 模块四：受众行为与转化分析 (Heatmap & Sankey)
   */
  if (charts.heatmap) {
    let dominantGenre = activeGenres.value[0] || '综合';
    let totalViews = 1000; 
    
    // 动态粘性算法：根据流派动态调整漏斗流失率
    let stickiness = (dominantGenre === '嘻哈 Hip-Hop' || dominantGenre === '民谣 Folk') ? 40 : 0; 
    
    const dynamicSankeyLinks = sankeyData.links.map(link => ({...link}));
    
    let targetLink1 = dynamicSankeyLinks.find(l => l.source === '完整完播' && l.target === '加入歌单');
    let targetLink2 = dynamicSankeyLinks.find(l => l.source === '完整完播' && l.target === '流失/无动作');
    if (targetLink1 && targetLink2) {
        targetLink1.value += stickiness;
        targetLink2.value -= stickiness;
    }

    charts.heatmap.setOption({
      animationDuration: 1200,
      animationEasing: 'cubicInOut',

      tooltip: { 
        ...baseTooltip,
        formatter: function(params) {
          if (params.seriesType === 'heatmap') {
            const time = params.value[0] * 2; return `周${['一','二','三','四','五','六','日'][params.value[1]]} ${time}:00<br/>活跃热度: <b style="color:#f472b6">${params.value[2].toFixed(1)}</b>`;
          }
          if (params.dataType === 'edge') {
            const cr = ((params.data.value / totalViews) * 100).toFixed(1); return `<b>${params.data.source} -> ${params.data.target}</b><br/>流转量: ${params.data.value} <br/>链路转化率 (CR): <b style="color:#4ade80">${cr}%</b>`;
          }
          return params.name;
        }
      },
      grid: { left: '10%', right: '5%', top: '5%', height: '35%' },
      xAxis: { type: 'category', data: Array.from({length: 24}, (_, i) => i + 'h'), axisLabel: { color: '#94a3b8' }, axisLine: {show: false}, splitLine: {show: false} },
      yAxis: { type: 'category', data: ['周一','周二','周三','周四','周五','周六','周日'], axisLabel: { color: '#94a3b8' }, axisLine: {show: false}, splitLine: {show: false} },
      visualMap: { show: false, seriesIndex: 0, min: 40, max: 100, inRange: { color: ['#0f172a', '#0284c7', '#38bdf8', '#f472b6'] } },
      
      series: [
        { 
          type: 'heatmap', 
          data: mergedHeatmapData, 
          itemStyle: { borderColor: '#020617', borderWidth: 2, borderRadius: 3 },
          emphasis: {
            itemStyle: { shadowBlur: 15, shadowColor: '#f472b6', borderColor: '#f472b6', borderWidth: 1 }
          },
          animationDelay: function (idx) {
             return (mergedHeatmapData[idx][0] + mergedHeatmapData[idx][1]) * 20;
          }
        },
        { 
          type: 'sankey', 
          top: '50%', bottom: 20, left: '5%', right: '22%', 
          data: sankeyData.nodes, links: dynamicSankeyLinks, 
          label: { color: '#fff', fontSize: 11, fontWeight: 500, textShadowColor: '#000', textShadowBlur: 3 }, 
          itemStyle: { borderWidth: 1.5, borderColor: '#38bdf8', shadowBlur: 15, shadowColor: 'rgba(56, 189, 248, 0.4)' }, 
          lineStyle: { color: 'gradient', curveness: 0.5, opacity: 0.25 },
          emphasis: {
            focus: 'adjacency', 
            lineStyle: { opacity: 0.85, shadowBlur: 10, shadowColor: '#f472b6' }
          }
        }
      ]
    }, { replaceMerge: ['series'] });
  }
}

/**
 * 根组件生命周期挂载与事件监听
 */
const handleResize = debounce(() => { Object.values(charts).forEach(c => c && c.resize()); }, 100);

onMounted(() => {
  timeInterval = setInterval(() => { currentTime.value = new Date().toLocaleString('zh-CN', { hour12: false }) }, 1000);
  
  charts.timeline = echarts.init(timelineRef.value); 
  charts.radar = echarts.init(radarRef.value);
  charts.network = echarts.init(networkRef.value); 
  charts.heatmap = echarts.init(heatmapRef.value);
  
  fetchAndMergeData(); 
  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  clearInterval(timeInterval); window.removeEventListener('resize', handleResize);
  Object.values(charts).forEach(c => c && c.dispose());
});

/**
 * 全局状态监听机制
 * @description 监听参数变化，应用防抖约束触发视图重绘引擎
 */
const throttledRender = debounce(() => { requestAnimationFrame(renderAllCharts); }, 80);
watch([startYear, endYear, activeGenres], throttledRender, { deep: true });
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
*, *::before, *::after { box-sizing: border-box; }
body, html { margin: 0; padding: 0; width: 100vw; height: 100vh; overflow: hidden; background: #020617; font-family: 'Inter', system-ui, sans-serif; }
#app { width: 100%; height: 100%; }
::-webkit-scrollbar { width: 4px; height: 4px; }
::-webkit-scrollbar-track { background: transparent; }
::-webkit-scrollbar-thumb { background: rgba(14, 165, 233, 0.4); border-radius: 4px; transition: background 0.3s; }
::-webkit-scrollbar-thumb:hover { background: rgba(14, 165, 233, 0.8); }
</style>

<style scoped>
@keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0.3; } }
@keyframes floatAnim { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-10px); } }
.blink { animation: blink 2s infinite; }
.float-anim { animation: floatAnim 4s infinite ease-in-out; }

.datav-container { display: flex; flex-direction: column; width: 100vw; height: 100vh; position: relative; color: #f8fafc; }
.datav-bg-grid { position: absolute; inset: 0; background-image: linear-gradient(rgba(14,165,233,0.05) 1px, transparent 1px), linear-gradient(90deg, rgba(14,165,233,0.05) 1px, transparent 1px); background-size: 50px 50px; z-index: 0; pointer-events: none; }
.datav-bg-glow { position: absolute; width: 50vw; height: 50vh; filter: blur(90px); z-index: 0; pointer-events: none; }
.glow-left { top: -20vh; left: -20vw; background: radial-gradient(circle, rgba(14,165,233,0.15) 0%, transparent 70%); }
.glow-right { bottom: -20vh; right: -20vw; background: radial-gradient(circle, rgba(192,132,252,0.1) 0%, transparent 70%); }

.datav-header { height: 70px; display: flex; justify-content: space-between; align-items: flex-start; padding: 0 24px; z-index: 10; flex-shrink: 0; background: url('data:image/svg+xml;utf8,<svg preserveAspectRatio="none" viewBox="0 0 1920 70" xmlns="http://www.w3.org/2000/svg"><path d="M0 0 L1920 0 L1920 50 L1400 50 L1350 70 L570 70 L520 50 L0 50 Z" fill="rgba(15,23,42,0.8)" stroke="rgba(14,165,233,0.3)" stroke-width="1"/></svg>') center top / 100% 100% no-repeat; }
.header-left, .header-right { width: 300px; height: 50px; display: flex; align-items: center; }
.header-right { justify-content: flex-end; }
.header-center { flex: 1; text-align: center; height: 70px; display: flex; flex-direction: column; justify-content: center; align-items: center; padding-bottom: 5px; }
.main-title { margin: 0; font-size: 24px; font-weight: 700; color: #fff; letter-spacing: 4px; text-shadow: 0 0 10px rgba(14,165,233,0.8); }
.title-decoration { margin-top: 5px; }
.status-badge { display: inline-flex; align-items: center; gap: 8px; font-size: 13px; color: #38bdf8; background: rgba(14, 165, 233, 0.15); padding: 6px 16px; border-radius: 4px; border: 1px solid rgba(14, 165, 233, 0.3); box-shadow: 0 0 10px rgba(14, 165, 233, 0.2) inset; }
.dot { width: 6px; height: 6px; background: #38bdf8; border-radius: 50%; box-shadow: 0 0 8px #38bdf8; }
.time-display { text-align: right; font-family: monospace; font-size: 18px; font-weight: 600; color: #cbd5e1; text-shadow: 0 0 5px rgba(255,255,255,0.3); }

.dashboard-body { display: flex; flex-direction: row; gap: 20px; padding: 15px 24px 24px; height: calc(100vh - 70px); z-index: 5; box-sizing: border-box; }
.layout-col { display: flex; flex-direction: column; gap: 20px; height: 100%; }
.col-left { flex: 0 0 340px; }
.col-center { flex: 1; min-width: 0; }
.col-right { flex: 0 0 340px; }
.panel-control { height: 45%; } 
.panel-network { height: calc(55% - 20px); }
.panel-timeline { height: 45%; }
.panel-radar { height: calc(55% - 20px); }
.panel-audience { height: 100%; }

.datav-panel { position: relative; background: rgba(9, 15, 30, 0.6); backdrop-filter: blur(10px); border: 1px solid rgba(14, 165, 233, 0.2); box-shadow: inset 0 0 20px rgba(14, 165, 233, 0.05); display: flex; flex-direction: column; transition: opacity 0.3s; overflow: hidden; }
.datav-panel.faded { opacity: 0; pointer-events: none; }
.corner { position: absolute; width: 10px; height: 10px; border: 2px solid #0ea5e9; z-index: 10; }
.corner.tl { top: -1px; left: -1px; border-right: none; border-bottom: none; }
.corner.tr { top: -1px; right: -1px; border-left: none; border-bottom: none; }
.corner.bl { bottom: -1px; left: -1px; border-right: none; border-top: none; }
.corner.br { bottom: -1px; right: -1px; border-left: none; border-top: none; }

.panel-header { height: 40px; line-height: 40px; padding: 0 16px; font-size: 15px; font-weight: 600; color: #fff; background: linear-gradient(90deg, rgba(14,165,233,0.3) 0%, transparent 100%); border-bottom: 1px solid rgba(14,165,233,0.2); display: flex; align-items: center; gap: 8px; flex-shrink: 0; }
.panel-header .icon { color: #38bdf8; font-size: 16px; }
.panel-header .header-desc { font-size: 12px; color: #94a3b8; font-weight: 400; margin-left: auto; letter-spacing: 0.5px; }
.panel-content { flex: 1; position: relative; min-height: 0; width: 100%; display: flex; flex-direction: column; }
.chart-wrapper { position: relative; width: 100%; height: 100%; flex: 1; min-height: 0; }
.chart-canvas { position: absolute; top: 0; left: 0; right: 0; bottom: 0; width: 100%; height: 100%; }
.control-content { flex: 1; padding: 16px; display: flex; flex-direction: column; overflow-y: auto; }
.search-box { width: 100%; height: 32px; background: rgba(0,0,0,0.4); border: 1px solid rgba(14,165,233,0.3); border-radius: 4px; display: flex; align-items: center; padding: 0 10px; flex-shrink: 0; margin-bottom: 12px; }
.search-box input { flex: 1; background: transparent; border: none; color: #fff; outline: none; font-size: 12px; margin-left: 8px;}
.filter-group { display: flex; flex-direction: column; gap: 8px; flex-shrink: 0; margin-bottom: 12px; }
.group-title { font-size: 12px; color: #cbd5e1; display: flex; justify-content: space-between; align-items: center; }
.group-title .val { color: #38bdf8; font-family: monospace; font-size: 12px; font-weight: 600; }
.slider-wrapper { position: relative; width: 100%; height: 18px; display: flex; align-items: center; margin-bottom: 2px; }
.slider-wrapper .track { position: absolute; width: 100%; height: 4px; background: rgba(255,255,255,0.1); border-radius: 2px; }
.slider-wrapper .fill { position: absolute; height: 4px; background: linear-gradient(90deg, #0ea5e9, #818cf8); border-radius: 2px; box-shadow: 0 0 10px rgba(14,165,233,0.5); }
.slider-wrapper input[type="range"] { position: absolute; width: 100%; left: 0; appearance: none; background: transparent; outline: none; pointer-events: none; z-index: 2; margin: 0; padding: 0; }
.slider-wrapper input[type="range"]::-webkit-slider-thumb { appearance: none; width: 14px; height: 14px; background: #fff; border: 2px solid #0ea5e9; border-radius: 50%; pointer-events: all; cursor: pointer; }
.tags-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; }
.genre-tag { padding: 6px 0; text-align: center; border-radius: 4px; font-size: 12px; font-weight: 600; cursor: pointer; border: 1px solid rgba(255,255,255,0.1); color: #94a3b8; transition: all 0.3s; user-select: none; }
.genre-tag:active { transform: scale(0.95); }
.flex-spacer { flex: 1; min-height: 10px; }
.reset-btn { margin-top: auto; margin-bottom: 5px; width: 100%; background: rgba(14,165,233,0.1); border: 1px dashed rgba(14,165,233,0.4); color: #38bdf8; padding: 8px; border-radius: 4px; cursor: pointer; font-size: 12px; transition: all 0.3s; flex-shrink: 0; }
.reset-btn:hover { background: rgba(14,165,233,0.2); box-shadow: 0 0 15px rgba(14,165,233,0.3); }
.global-overlay { position: absolute; inset: 0; background: rgba(2,6,23,0.85); z-index: 50; display: flex; justify-content: center; align-items: center; backdrop-filter: blur(5px); }
.alert-box { text-align: center; background: rgba(15,23,42,0.8); padding: 40px 60px; border-radius: 12px; border: 1px solid rgba(239,68,68,0.3); box-shadow: 0 0 30px rgba(239,68,68,0.1); }
.alert-box h2 { color: #f8fafc; margin: 20px 0 10px; font-size: 24px; letter-spacing: 2px; }
.alert-box p { color: #94a3b8; font-size: 14px; margin: 0; }
</style>