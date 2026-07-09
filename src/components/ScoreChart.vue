<template>
  <div class="chart-wrapper" ref="wrapperRef">
    <apexchart
      type="bar"
      :height="chartHeight"
      :options="chartOptions"
      :series="series"
    />
    <div class="logos-overlay" aria-hidden="true">
      <div
        v-for="(pos, i) in barPositions"
        :key="i"
        class="bar-logo"
        :style="{
          left: pos.cx - pos.size / 2 + 'px',
          top: pos.logoTop + 'px',
          width: pos.size + 'px',
          height: pos.size + 'px',
        }"
      >
        <img :src="props.teams[i].logo" :alt="props.teams[i].name" />
      </div>
      <div
        v-for="(pos, i) in barPositions"
        :key="'score-' + i"
        class="bar-score"
        :style="{
          left: pos.cx + 'px',
          top: pos.scoreTop + 'px',
          color: '#ffffff',
        }"
      >
        {{ props.teams[i].score > 0 ? props.teams[i].score.toLocaleString('pt-BR') : '' }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref, nextTick } from 'vue'
import VueApexCharts from 'vue3-apexcharts'

const apexchart = VueApexCharts

const props = defineProps({
  teams: {
    type: Array,
    required: true,
  },
})

const wrapperRef = ref(null)
const barPositions = ref([])

const isMobile = () => window.innerWidth < 640

const chartHeight = computed(() => (isMobile() ? 320 : 460))

const series = computed(() => [
  {
    name: 'Pontuação',
    data: props.teams.map((t) => t.score),
  },
])

function measureBars(chartEl) {
  if (!wrapperRef.value) return

  const bars = chartEl.querySelectorAll(
    '.apexcharts-bar-series .apexcharts-series path, .apexcharts-bar-series .apexcharts-series rect'
  )
  if (!bars.length) return

  const wrapperRect = wrapperRef.value.getBoundingClientRect()
  const mobile = isMobile()
  const positions = []

  bars.forEach((bar, i) => {
    if (i >= props.teams.length) return
    const r = bar.getBoundingClientRect()
    if (r.width === 0 && r.height === 0) return

    // coordinates relative to the wrapper div
    const cx = r.left - wrapperRect.left + r.width / 2
    const barVisualTop = r.top - wrapperRect.top      // top edge of bar (visually highest)

    const size = Math.min(Math.max(r.width * 0.65, 24), 52)

    let logoTop, scoreTop
    if (mobile) {
      // logo inside bar, near the top
      logoTop = barVisualTop + 6
      scoreTop = barVisualTop - 22
    } else {
      // logo floating above bar
      logoTop = barVisualTop - size - 8
      scoreTop = barVisualTop - size - 8 - 22
    }

    positions.push({ cx, barVisualTop, logoTop, scoreTop, size })
  })

  barPositions.value = positions
}

function onChartReady(chartContext) {
  nextTick(() => measureBars(chartContext.el))
}

function onChartUpdated(chartContext) {
  setTimeout(() => measureBars(chartContext.el), 550)
}

const chartOptions = computed(() => ({
  chart: {
    type: 'bar',
    background: 'transparent',
    toolbar: { show: false },
    animations: {
      enabled: true,
      easing: 'easeinout',
      speed: 900,
      animateGradually: { enabled: true, delay: 150 },
      dynamicAnimation: { enabled: true, speed: 500 },
    },
    events: {
      mounted: onChartReady,
      updated: onChartUpdated,
    },
  },
  plotOptions: {
    bar: {
      borderRadius: 12,
      borderRadiusApplication: 'end',
      columnWidth: '55%',
      distributed: true,
      dataLabels: { position: 'top' },
    },
  },
  colors: props.teams.map((t) => t.primaryColor),
  fill: {
    type: 'gradient',
    gradient: {
      shade: 'dark',
      type: 'vertical',
      shadeIntensity: 0.5,
      gradientToColors: props.teams.map((t) => t.gradient[1]),
      inverseColors: false,
      opacityFrom: 1,
      opacityTo: 0.85,
      stops: [0, 100],
    },
  },
  dataLabels: { enabled: false },
  xaxis: {
    categories: props.teams.map((t) => t.name),
    labels: { show: false },
    axisBorder: { show: false },
    axisTicks: { show: false },
  },
  yaxis: {
    labels: {
      style: {
        colors: '#718096',
        fontSize: '13px',
        fontFamily: 'Inter, sans-serif',
      },
      formatter: (val) => val.toLocaleString('pt-BR'),
    },
  },
  grid: {
    borderColor: '#2d3748',
    strokeDashArray: 4,
    xaxis: { lines: { show: false } },
    yaxis: { lines: { show: true } },
  },
  tooltip: {
    theme: 'dark',
    y: {
      formatter: (val) => `${val.toLocaleString('pt-BR')} pts`,
    },
    style: {
      fontSize: '14px',
      fontFamily: 'Inter, sans-serif',
    },
  },
  legend: { show: false },
  states: {
    hover: { filter: { type: 'lighten', value: 0.1 } },
    active: { filter: { type: 'none' } },
  },
}))
</script>

<style scoped>
.chart-wrapper {
  width: 100%;
  position: relative;
}

.logos-overlay {
  position: absolute;
  inset: 0;
  pointer-events: none;
}

.bar-logo {
  position: absolute;
  border-radius: 50%;
  background: rgba(10, 14, 26, 0.55);
  padding: 3px;
  display: flex;
  align-items: center;
  justify-content: center;
  transform: translateX(0);
}

.bar-logo img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 50%;
}

.bar-score {
  position: absolute;
  transform: translateX(-50%);
  font-size: 14px;
  font-weight: 700;
  font-family: 'Inter', sans-serif;
  text-shadow: 0 1px 4px rgba(0, 0, 0, 0.9);
  white-space: nowrap;
}
</style>
