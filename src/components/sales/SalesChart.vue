<template>
  <div class="sales-chart p-4 border-round-xl surface-card">
    <h3 class="text-xl font-semibold mb-4">{{ title }}</h3>
    <canvas ref="chartRef"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';
import { Chart } from 'chart.js/auto';

const props = defineProps({
  title: {
    type: String,
    required: true
  },
  type: {
    type: String,
    required: true,
    validator: (value) => ['bar', 'pie'].includes(value)
  },
  data: {
    type: Object,
    required: true
  }
});

const chartRef = ref(null);
let chartInstance = null;

const createChart = () => {
  if (chartInstance) {
    chartInstance.destroy();
  }

  const ctx = chartRef.value.getContext('2d');
  chartInstance = new Chart(ctx, {
    type: props.type,
    data: props.data,
    options: {
      responsive: true,
      plugins: {
        legend: {
          display: props.type === 'pie',
          position: 'bottom'
        }
      },
      scales: props.type === 'bar' ? {
        y: {
          beginAtZero: true,
          ticks: {
            callback: value => `$${value}`
          }
        }
      } : undefined
    }
  });
};

onMounted(createChart);
watch(() => props.data, createChart, { deep: true });
</script>

<style scoped>
.sales-chart {
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(10px);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  height: 100%;
}
</style>