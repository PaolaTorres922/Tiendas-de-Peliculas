<template>
  <div class="sales-card p-4 border-round-xl surface-card">
    <div class="flex align-items-center mb-3">
      <i :class="[icon, 'text-xl text-primary mr-2']"></i>
      <span class="text-900 font-medium">{{ title }}</span>
    </div>
    <div class="text-2xl font-bold text-900">{{ value }}</div>
  </div>
</template>

<script setup>
defineProps({
  title: {
    type: String,
    required: true
  },
  value: {
    type: [String, Number],
    required: true
  },
  icon: {
    type: String,
    required: true
  }
});
</script>

<style scoped>
.sales-card {
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(10px);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.sales-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}
</style>