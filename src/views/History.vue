<template>
  <div class="history-view">
    <div class="hero-section">
      <h1 class="text-4xl font-bold mb-2">Historial de Transacciones</h1>
      <p class="text-xl opacity-90">Revisa todas tus compras anteriores</p>
    </div>

    <div class="container">
      <Timeline :value="timelineEvents" class="mt-4">
        <template #content="slotProps">
          <Card class="mb-3">
            <template #title>
              <div class="flex align-items-center gap-2">
                <i class="pi pi-shopping-cart text-primary"></i>
                <span>{{ slotProps.item.movieTitle }}</span>
              </div>
            </template>
            <template #subtitle>
              {{ formatDate(slotProps.item.date) }}
            </template>
            <template #content>
              <div class="grid">
                <div class="col-12 md:col-4">
                  <div class="flex align-items-center gap-2">
                    <i class="pi pi-tag"></i>
                    <span>Cantidad: {{ slotProps.item.quantity }}</span>
                  </div>
                </div>
                <div class="col-12 md:col-4">
                  <div class="flex align-items-center gap-2">
                    <i class="pi pi-percentage"></i>
                    <span>Descuento: {{ slotProps.item.discount }}%</span>
                  </div>
                </div>
                <div class="col-12 md:col-4">
                  <div class="flex align-items-center gap-2">
                    <i class="pi pi-money-bill"></i>
                    <span class="text-green-600 font-bold">
                      Total: ${{ slotProps.item.finalPrice.toFixed(2) }}
                    </span>
                  </div>
                </div>
              </div>
            </template>
          </Card>
        </template>
      </Timeline>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue';
import { useMovieStore } from '../stores/movies';
import Timeline from 'primevue/timeline';
import Card from 'primevue/card';

const movieStore = useMovieStore();

const timelineEvents = computed(() => {
  return movieStore.sales.map(sale => {
    const movie = movieStore.movieById(sale.movieId);
    const originalPrice = sale.price * sale.quantity;
    const finalPrice = originalPrice * (1 - (sale.discount / 100));
    
    return {
      ...sale,
      movieTitle: movie ? movie.title : 'Película no encontrada',
      originalPrice,
      finalPrice,
      date: new Date(sale.date)
    };
  }).sort((a, b) => b.date - a.date);
});

const formatDate = (date) => {
  return date.toLocaleDateString('es-ES', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  });
};
</script>

<style scoped>
.history-view {
  min-height: calc(100vh - 4rem);
  background: linear-gradient(135deg, var(--surface-ground) 0%, var(--surface-section) 100%);
}

.hero-section {
  background: linear-gradient(135deg, var(--primary-color), var(--primary-color-darker));
  color: white;
  padding: 3rem 2rem;
  text-align: center;
  margin-bottom: 2rem;
  animation: fadeIn 0.5s ease-out;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
}

:deep(.p-timeline) {
  padding: 2rem;
}

:deep(.p-timeline-event-content) {
  background: rgba(255, 255, 255, 0.9);
  border-radius: 1rem;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease;
}

:deep(.p-timeline-event-content:hover) {
  transform: translateY(-5px);
}

:deep(.p-timeline-event-opposite) {
  display: none;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media screen and (max-width: 768px) {
  .hero-section {
    padding: 2rem 1rem;
  }
  
  :deep(.p-timeline) {
    padding: 1rem;
  }
}
</style>