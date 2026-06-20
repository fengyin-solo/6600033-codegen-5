<template>
  <div class="min-h-screen bg-slate-900 text-slate-200">
    <header class="border-b border-slate-700 px-6 py-4">
      <h1 class="text-2xl font-bold text-cyan-400">蒙特卡洛模拟与统计假设检验平台</h1>
      <p class="text-sm text-slate-500 mt-1">随机采样模拟 · 6种MC场景 · 假设检验 · 置信区间可视化</p>
    </header>
    <div class="flex flex-col lg:flex-row gap-4 p-4">
      <div class="lg:w-1/4 space-y-4">
        <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
          <h3 class="text-sm font-bold text-slate-400 mb-3">模拟场景</h3>
          <div class="space-y-1">
            <div v-for="s in SCENARIOS" :key="s.id" @click="store.setScenario(s)"
              :class="['cursor-pointer p-2 rounded border text-sm transition-all', store.currentScenario.id === s.id ? 'border-cyan-500 bg-cyan-900/30 text-cyan-400' : 'border-slate-700 text-slate-300 hover:border-slate-500']">
              <div class="font-bold">{{ s.name }}</div>
              <div class="text-xs text-slate-500 mt-0.5">{{ s.description }}</div>
            </div>
          </div>
        </div>
        <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
          <h3 class="text-sm font-bold text-slate-400 mb-3">参数控制</h3>
          <label class="text-xs text-slate-500">迭代次数: {{ store.iterations }}</label>
          <input type="range" min="100" max="5000" step="100" v-model.number="store.iterations" class="w-full mt-1 mb-3 accent-cyan-500" />
          <button @click="store.runSimulation" :disabled="store.isRunning" class="w-full py-2 bg-cyan-600 hover:bg-cyan-500 disabled:opacity-50 rounded text-sm font-bold">
            {{ store.isRunning ? '运行中...' : '▶ 开始模拟' }}
          </button>
        </div>
        <div v-if="store.result" class="bg-slate-800 rounded-lg p-4 border border-slate-700 text-sm">
          <h3 class="text-sm font-bold text-slate-400 mb-3">模拟结果</h3>
          <div class="space-y-2">
            <div class="flex justify-between"><span class="text-slate-500">估算值</span><span class="text-cyan-400 font-bold font-mono">{{ store.result.estimate.toFixed(6) }}</span></div>
            <div v-if="store.result.trueValue !== undefined" class="flex justify-between"><span class="text-slate-500">真实值</span><span class="text-green-400 font-mono">{{ store.result.trueValue.toFixed(6) }}</span></div>
            <div v-if="store.result.error !== undefined" class="flex justify-between"><span class="text-slate-500">误差</span><span class="text-orange-400 font-mono">{{ store.result.error.toFixed(6) }}</span></div>
            <div class="flex justify-between"><span class="text-slate-500">样本数</span><span class="text-slate-300">{{ store.result.iterations }}</span></div>
          </div>
        </div>
      </div>
      <div class="lg:w-3/4 space-y-4">
        <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
          <h3 class="text-sm font-bold text-slate-400 mb-3">收敛过程</h3>
          <div ref="convergenceRef" class="w-full rounded" style="height:240px;background:#0f172a;"></div>
        </div>
        <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
          <h3 class="text-sm font-bold text-slate-400 mb-3">样本分布直方图</h3>
          <div ref="histogramRef" class="w-full rounded" style="height:220px;background:#0f172a;"></div>
        </div>
        <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
          <h3 class="text-sm font-bold text-slate-400 mb-3">假设检验 (独立样本 T 检验)</h3>
          <div class="grid grid-cols-2 gap-4 mb-3">
            <div>
              <label class="text-xs text-slate-500">样本组A (逗号分隔)</label>
              <textarea v-model="group1Input" rows="2" class="w-full mt-1 bg-slate-900 border border-slate-600 rounded px-2 py-1 text-xs font-mono focus:outline-none focus:border-cyan-500 resize-none"></textarea>
            </div>
            <div>
              <label class="text-xs text-slate-500">样本组B (逗号分隔)</label>
              <textarea v-model="group2Input" rows="2" class="w-full mt-1 bg-slate-900 border border-slate-600 rounded px-2 py-1 text-xs font-mono focus:outline-none focus:border-cyan-500 resize-none"></textarea>
            </div>
          </div>
          <button @click="runTest" class="px-4 py-1.5 bg-purple-600 hover:bg-purple-500 rounded text-sm">执行T检验</button>
          <div v-if="store.testResult" class="mt-3 grid grid-cols-4 gap-3 text-sm">
            <div class="bg-slate-900 rounded p-2 text-center"><div class="text-xs text-slate-500 mb-1">统计量 t</div><div class="text-cyan-400 font-bold font-mono">{{ store.testResult.statistic }}</div></div>
            <div class="bg-slate-900 rounded p-2 text-center"><div class="text-xs text-slate-500 mb-1">p 值</div><div class="font-bold font-mono" :class="store.testResult.significant ? 'text-red-400' : 'text-green-400'">{{ store.testResult.pValue }}</div></div>
            <div class="bg-slate-900 rounded p-2 text-center"><div class="text-xs text-slate-500 mb-1">自由度 df</div><div class="text-slate-300 font-mono">{{ store.testResult.df }}</div></div>
            <div class="bg-slate-900 rounded p-2 text-center"><div class="text-xs text-slate-500 mb-1">显著性</div><div class="text-xs font-bold" :class="store.testResult.significant ? 'text-red-400' : 'text-green-400'">{{ store.testResult.significant ? '显著(p<0.05)' : '不显著' }}</div></div>
          </div>
        </div>
        <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
          <div class="flex items-center justify-between mb-3">
            <h3 class="text-sm font-bold text-slate-400">多方案置信区间对比</h3>
            <div class="flex gap-2">
              <select v-model.number="confidenceLevel" @change="onConfidenceLevelChange" class="bg-slate-900 border border-slate-600 rounded px-2 py-1 text-xs text-slate-300 focus:outline-none focus:border-cyan-500">
                <option :value="0.9">90% 置信度</option>
                <option :value="0.95">95% 置信度</option>
                <option :value="0.99">99% 置信度</option>
              </select>
              <button @click="addToComparison" :disabled="!store.result" class="px-3 py-1 bg-cyan-600 hover:bg-cyan-500 disabled:opacity-50 disabled:cursor-not-allowed rounded text-xs font-bold">+ 加入对比</button>
              <button @click="clearComparison" :disabled="store.comparisonScenarios.length === 0" class="px-3 py-1 bg-slate-600 hover:bg-slate-500 disabled:opacity-50 disabled:cursor-not-allowed rounded text-xs">清空</button>
            </div>
          </div>
          <div ref="ciComparisonRef" class="w-full rounded mb-3" style="height:280px;background:#0f172a;"></div>
          <div v-if="store.comparisonScenarios.length > 0" class="space-y-2">
            <div class="text-xs text-slate-500 font-medium mb-1">方案列表</div>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-2">
              <div v-for="s in store.comparisonScenarios" :key="s.id" class="bg-slate-900 rounded p-3 border border-slate-700">
                <div class="flex items-center justify-between mb-2">
                  <div class="flex items-center gap-2">
                    <div class="w-3 h-3 rounded-full" :style="{ backgroundColor: s.color }"></div>
                    <span class="text-sm font-medium text-slate-200">{{ s.name }}</span>
                  </div>
                  <button @click="removeFromComparison(s.id)" class="text-xs text-slate-500 hover:text-red-400">✕</button>
                </div>
                <div class="grid grid-cols-3 gap-2 text-xs">
                  <div>
                    <div class="text-slate-500">均值</div>
                    <div class="font-mono text-slate-300">{{ s.confidenceInterval.mean.toFixed(4) }}</div>
                  </div>
                  <div>
                    <div class="text-slate-500">下限</div>
                    <div class="font-mono text-slate-300">{{ s.confidenceInterval.lower.toFixed(4) }}</div>
                  </div>
                  <div>
                    <div class="text-slate-500">上限</div>
                    <div class="font-mono text-slate-300">{{ s.confidenceInterval.upper.toFixed(4) }}</div>
                  </div>
                </div>
                <div class="mt-2 text-xs">
                  <div class="text-slate-500">区间宽度</div>
                  <div class="font-mono text-cyan-400">{{ (s.confidenceInterval.upper - s.confidenceInterval.lower).toFixed(4) }}</div>
                </div>
              </div>
            </div>
            <div v-if="store.comparisonScenarios.length >= 2" class="mt-3 pt-3 border-t border-slate-700">
              <div class="text-xs text-slate-500 font-medium mb-2">区间重叠分析</div>
              <div class="space-y-1">
                <div v-for="pair in overlapPairs" :key="pair.key" class="flex items-center justify-between text-xs bg-slate-900 rounded px-3 py-2">
                  <div class="flex items-center gap-2">
                    <div class="w-2 h-2 rounded-full" :style="{ backgroundColor: pair.color1 }"></div>
                    <span class="text-slate-400">{{ pair.name1 }}</span>
                    <span class="text-slate-600">vs</span>
                    <div class="w-2 h-2 rounded-full" :style="{ backgroundColor: pair.color2 }"></div>
                    <span class="text-slate-400">{{ pair.name2 }}</span>
                  </div>
                  <span :class="['font-bold', pair.overlaps ? 'text-green-400' : 'text-red-400']">
                    {{ pair.overlaps ? '区间重叠' : '区间不重叠' }}
                  </span>
                </div>
              </div>
            </div>
          </div>
          <div v-else class="text-center py-8 text-slate-500 text-sm">
            <div class="text-3xl mb-2">📊</div>
            <div>暂无对比方案</div>
            <div class="text-xs mt-1">点击"加入对比"将当前模拟结果添加到对比视图</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'
import * as echarts from 'echarts'
import { useMCStore, SCENARIOS, type ComparisonScenario } from './store/mc'

const store = useMCStore()
const convergenceRef = ref<HTMLDivElement | null>(null)
const histogramRef = ref<HTMLDivElement | null>(null)
const ciComparisonRef = ref<HTMLDivElement | null>(null)
const group1Input = ref('5.1,4.8,5.3,4.9,5.2,5.0,4.7,5.1,5.4,4.8')
const group2Input = ref('4.6,4.2,4.9,4.3,4.5,4.7,4.4,4.8,4.1,4.6')
const confidenceLevel = ref(0.95)
let convChart: echarts.ECharts | null = null
let histChart: echarts.ECharts | null = null
let ciChart: echarts.ECharts | null = null

const overlapPairs = computed(() => {
  const pairs: Array<{ key: string; name1: string; name2: string; color1: string; color2: string; overlaps: boolean }> = []
  const scenarios = store.comparisonScenarios
  for (let i = 0; i < scenarios.length; i++) {
    for (let j = i + 1; j < scenarios.length; j++) {
      pairs.push({
        key: `${scenarios[i].id}-${scenarios[j].id}`,
        name1: scenarios[i].name,
        name2: scenarios[j].name,
        color1: scenarios[i].color,
        color2: scenarios[j].color,
        overlaps: store.checkOverlap(scenarios[i], scenarios[j])
      })
    }
  }
  return pairs
})

function initCharts() {
  if (convergenceRef.value) convChart = echarts.init(convergenceRef.value, 'dark')
  if (histogramRef.value) histChart = echarts.init(histogramRef.value, 'dark')
  if (ciComparisonRef.value) ciChart = echarts.init(ciComparisonRef.value, 'dark')
}

function updateCharts() {
  if (convChart && store.convergenceData.length > 0) {
    convChart.setOption({
      backgroundColor: '#0f172a',
      grid: { top: 20, bottom: 35, left: 65, right: 20 },
      xAxis: { type: 'value', axisLabel: { color: '#94a3b8', fontSize: 10 } },
      yAxis: { type: 'value', axisLabel: { color: '#94a3b8', fontSize: 10 } },
      series: [{ type: 'line', data: store.convergenceData, smooth: true, lineStyle: { color: '#06b6d4', width: 2 }, areaStyle: { color: 'rgba(6,182,212,0.1)' }, symbol: 'none' }],
      tooltip: { trigger: 'axis', backgroundColor: '#1e293b', borderColor: '#475569' }
    })
  }
  if (histChart && store.histogramData.xAxis.length > 0) {
    histChart.setOption({
      backgroundColor: '#0f172a',
      grid: { top: 15, bottom: 40, left: 55, right: 15 },
      xAxis: { type: 'category', data: store.histogramData.xAxis, axisLabel: { color: '#94a3b8', fontSize: 9, rotate: 30 } },
      yAxis: { type: 'value', axisLabel: { color: '#94a3b8', fontSize: 10 } },
      series: [{ type: 'bar', data: store.histogramData.data, itemStyle: { color: '#8b5cf6' } }],
      tooltip: { trigger: 'axis', backgroundColor: '#1e293b', borderColor: '#475569' }
    })
  }
  updateCIChart()
}

function updateCIChart() {
  if (!ciChart) return
  const scenarios = store.comparisonScenarios
  if (scenarios.length === 0) {
    ciChart.clear()
    return
  }
  const categories = scenarios.map(s => s.name)

  const allValues = scenarios.flatMap(s => [s.confidenceInterval.lower, s.confidenceInterval.upper])
  const minVal = Math.min(...allValues)
  const maxVal = Math.max(...allValues)
  const padding = (maxVal - minVal) * 0.1 || Math.abs(minVal) * 0.1 || 0.1

  ciChart.setOption({
    backgroundColor: '#0f172a',
    grid: { top: 25, bottom: 45, left: 140, right: 30 },
    xAxis: {
      type: 'value',
      min: minVal - padding,
      max: maxVal + padding,
      axisLabel: { color: '#94a3b8', fontSize: 10 },
      splitLine: { lineStyle: { color: '#1e293b' } }
    },
    yAxis: {
      type: 'category',
      data: categories,
      axisLabel: { color: '#e2e8f0', fontSize: 11, fontWeight: 500 },
      axisLine: { lineStyle: { color: '#334155' } },
      axisTick: { show: false },
      splitLine: { show: false }
    },
    tooltip: {
      trigger: 'item',
      backgroundColor: '#1e293b',
      borderColor: '#475569',
      textStyle: { color: '#e2e8f0', fontSize: 12 },
      formatter: (params: any) => {
        const scenario = scenarios[params.dataIndex]
        if (!scenario) return ''
        const ci = scenario.confidenceInterval
        return `<div style="min-width:180px;">
          <div style="font-weight:bold;margin-bottom:6px;color:${scenario.color};font-size:13px;">${scenario.name}</div>
          <div style="display:flex;justify-content:space-between;margin-bottom:3px;">
            <span style="color:#94a3b8;">均值</span>
            <span style="font-family:monospace;color:#e2e8f0;">${ci.mean.toFixed(4)}</span>
          </div>
          <div style="display:flex;justify-content:space-between;margin-bottom:3px;">
            <span style="color:#94a3b8;">置信区间</span>
            <span style="font-family:monospace;color:#06b6d4;">[${ci.lower.toFixed(4)}, ${ci.upper.toFixed(4)}]</span>
          </div>
          <div style="display:flex;justify-content:space-between;margin-bottom:3px;">
            <span style="color:#94a3b8;">区间宽度</span>
            <span style="font-family:monospace;color:#e2e8f0;">${(ci.upper - ci.lower).toFixed(4)}</span>
          </div>
          <div style="display:flex;justify-content:space-between;">
            <span style="color:#94a3b8;">样本数</span>
            <span style="font-family:monospace;color:#e2e8f0;">${scenario.iterations}</span>
          </div>
        </div>`
      }
    },
    series: [{
      type: 'custom',
      renderItem: (params: any, api: any) => {
        const idx = params.dataIndex
        const s = scenarios[idx]
        const ci = s.confidenceInterval
        const barHeight = 24

        const lowerPoint = api.coord([ci.lower, idx])
        const upperPoint = api.coord([ci.upper, idx])
        const meanPoint = api.coord([ci.mean, idx])

        const yCenter = lowerPoint[1]
        const yTop = yCenter - barHeight / 2
        const yBottom = yCenter + barHeight / 2

        return {
          type: 'group',
          children: [
            {
              type: 'rect',
              shape: {
                x: lowerPoint[0],
                y: yTop,
                width: upperPoint[0] - lowerPoint[0],
                height: barHeight
              },
              style: {
                fill: s.color,
                opacity: 0.3
              }
            },
            {
              type: 'line',
              shape: {
                x1: lowerPoint[0],
                y1: yCenter,
                x2: upperPoint[0],
                y2: yCenter
              },
              style: {
                stroke: s.color,
                lineWidth: 2
              }
            },
            {
              type: 'line',
              shape: {
                x1: lowerPoint[0],
                y1: yTop,
                x2: lowerPoint[0],
                y2: yBottom
              },
              style: {
                stroke: s.color,
                lineWidth: 2
              }
            },
            {
              type: 'line',
              shape: {
                x1: upperPoint[0],
                y1: yTop,
                x2: upperPoint[0],
                y2: yBottom
              },
              style: {
                stroke: s.color,
                lineWidth: 2
              }
            },
            {
              type: 'circle',
              shape: {
                cx: meanPoint[0],
                cy: yCenter,
                r: 6
              },
              style: {
                fill: s.color,
                stroke: '#ffffff',
                lineWidth: 2
              }
            }
          ]
        }
      },
      data: scenarios.map((s) => s.confidenceInterval.mean),
      clip: true
    }]
  })
}

function runTest() {
  const g1 = group1Input.value.split(',').map(s => parseFloat(s.trim())).filter(n => !isNaN(n))
  const g2 = group2Input.value.split(',').map(s => parseFloat(s.trim())).filter(n => !isNaN(n))
  if (g1.length > 1 && g2.length > 1) store.runTest(g1, g2)
}

function addToComparison() {
  store.addCurrentToComparison()
}

function removeFromComparison(id: string) {
  store.removeFromComparison(id)
}

function clearComparison() {
  store.clearComparison()
}

function onConfidenceLevelChange() {
  store.setConfidenceLevel(confidenceLevel.value)
}

function handleResize() {
  convChart?.resize()
  histChart?.resize()
  ciChart?.resize()
}

onMounted(() => {
  initCharts()
  store.runSimulation()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  convChart?.dispose()
  histChart?.dispose()
  ciChart?.dispose()
})

watch(() => store.result, () => updateCharts(), { deep: true })
watch(() => store.comparisonScenarios, () => updateCIChart(), { deep: true })
</script>
