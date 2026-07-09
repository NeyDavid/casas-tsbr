<template>
  <div class="chart-wrapper">
    <apexchart
      type="bar"
      :height="chartHeight"
      :options="chartOptions"
      :series="series"
    />
  </div>
</template>

<script setup>
import { computed } from 'vue'
import VueApexCharts from 'vue3-apexcharts'

const apexchart = VueApexCharts

const props = defineProps({
  teams: {
    type: Array,
    required: true,
  },
})

const chartHeight = computed(() => {
  return window.innerWidth < 640 ? 320 : 460
})

const series = computed(() => [
  {
    name: 'Pontuação',
    data: props.teams.map((t) => t.score),
  },
])

function injectLogos(chartContext) {
  const svg = chartContext.el.querySelector('svg')
  if (!svg) return

  svg.querySelectorAll('.injected-team-logo').forEach((el) => el.remove())

  const bars = chartContext.el.querySelectorAll(
    '.apexcharts-bar-series .apexcharts-series path, .apexcharts-bar-series .apexcharts-series rect'
  )

  bars.forEach((bar, i) => {
    const team = props.teams[i]
    if (!team) return

    const bbox = bar.getBBox()
    if (bbox.width === 0 && bbox.height === 0) return

    const size = Math.min(Math.max(bbox.width * 0.65, 24), 52)
    const cx = bbox.x + bbox.width / 2
    // logo centered just above bar top
    const cy = bbox.y - size / 2 - 6

    const circle = document.createElementNS('http://www.w3.org/2000/svg', 'circle')
    circle.setAttributeNS(null, 'cx', cx)
    circle.setAttributeNS(null, 'cy', cy)
    circle.setAttributeNS(null, 'r', size / 2 + 3)
    circle.setAttributeNS(null, 'fill', 'rgba(10,14,26,0.7)')
    circle.setAttributeNS(null, 'class', 'injected-team-logo')
    svg.appendChild(circle)

    const img = document.createElementNS('http://www.w3.org/2000/svg', 'image')
    img.setAttributeNS(null, 'href', team.logo)
    img.setAttributeNS(null, 'x', cx - size / 2)
    img.setAttributeNS(null, 'y', cy - size / 2)
    img.setAttributeNS(null, 'width', size)
    img.setAttributeNS(null, 'height', size)
    img.setAttributeNS(null, 'class', 'injected-team-logo')
    svg.appendChild(img)

    if (team.score > 0) {
      const label = document.createElementNS('http://www.w3.org/2000/svg', 'text')
      label.setAttributeNS(null, 'x', cx)
      // position score text above the logo circle
      label.setAttributeNS(null, 'y', cy - size / 2 - 10)
      label.setAttributeNS(null, 'text-anchor', 'middle')
      label.setAttributeNS(null, 'dominant-baseline', 'auto')
      label.setAttributeNS(null, 'fill', '#ffffff')
      label.setAttributeNS(null, 'font-size', '15')
      label.setAttributeNS(null, 'font-weight', '700')
      label.setAttributeNS(null, 'font-family', 'Inter, sans-serif')
      label.setAttributeNS(null, 'filter', 'drop-shadow(0 1px 3px rgba(0,0,0,0.8))')
      label.setAttributeNS(null, 'class', 'injected-team-logo')
      label.textContent = team.score.toLocaleString('pt-BR')
      svg.appendChild(label)
    }
  })
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
      mounted: (chartContext) => injectLogos(chartContext),
      updated: (chartContext) => {
        setTimeout(() => injectLogos(chartContext), 520)
      },
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
    labels: {
      style: {
        colors: '#a0aec0',
        fontSize: '14px',
        fontFamily: 'Inter, sans-serif',
        fontWeight: '600',
      },
    },
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
}
</style>
