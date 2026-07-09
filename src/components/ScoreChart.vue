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
  dataLabels: {
    enabled: true,
    formatter: (val) => (val > 0 ? val.toLocaleString('pt-BR') : ''),
    offsetY: -12,
    style: {
      fontSize: '15px',
      fontFamily: 'Inter, sans-serif',
      fontWeight: '700',
      colors: ['#ffffff'],
    },
    dropShadow: {
      enabled: true,
      top: 1,
      left: 0,
      blur: 4,
      color: '#000',
      opacity: 0.6,
    },
  },
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
