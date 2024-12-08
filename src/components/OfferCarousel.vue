<template>
  <div class="offers-section">
    <h2 class="section-title">
      <i class="pi pi-tag mr-2"></i>
      Ofertas Especiales
    </h2>
    
    <Carousel 
      :value="movies" 
      :numVisible="4" 
      :numScroll="1" 
      :responsiveOptions="responsiveOptions"
      class="custom-carousel"
      :autoplayInterval="3000"
      :circular="true"
    >
      <template #item="slotProps">
        <div class="movie-offer-card">
          <div class="relative">
            <img :src="slotProps.data.image" :alt="slotProps.data.title" 
                 class="w-full h-15rem object-cover rounded-lg shadow-lg" />
            <div class="offer-overlay">
              <span class="discount-pill">-{{ slotProps.data.discount }}%</span>
              <div class="offer-details">
                <h3 class="text-xl font-bold text-white mb-2">{{ slotProps.data.title }}</h3>
                <div class="price-container">
                  <span class="original-price">${{ slotProps.data.price }}</span>
                  <span class="final-price">${{ calculateDiscountedPrice(slotProps.data) }}</span>
                </div>
                <Button 
                  :disabled="slotProps.data.stock === 0"
                  @click="$emit('purchase', slotProps.data)"
                  :label="slotProps.data.stock > 0 ? 'Comprar Ahora' : 'Sin Stock'"
                  class="p-button-rounded p-button-lg mt-3"
                />
              </div>
            </div>
          </div>
        </div>
      </template>
    </Carousel>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import Carousel from 'primevue/carousel';
import Button from 'primevue/button';

const props = defineProps({
  movies: {
    type: Array,
    required: true
  }
});

defineEmits(['purchase']);

const responsiveOptions = ref([
  {
    breakpoint: '1400px',
    numVisible: 3,
    numScroll: 1
  },
  {
    breakpoint: '1199px',
    numVisible: 2,
    numScroll: 1
  },
  {
    breakpoint: '767px',
    numVisible: 1,
    numScroll: 1
  }
]);

const calculateDiscountedPrice = (movie) => {
  return (movie.price * (1 - movie.discount / 100)).toFixed(2);
};
</script>

<style scoped>
.movie-offer-card {
  padding: 1rem;
  transition: transform 0.3s ease;
}

.movie-offer-card:hover {
  transform: scale(1.05);
}

.offer-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(to top, rgba(0,0,0,0.8) 0%, transparent 100%);
  padding: 2rem 1rem 1rem;
  border-radius: 0 0 0.5rem 0.5rem;
  opacity: 0;
  transform: translateY(20px);
  transition: all 0.3s ease;
}

.movie-offer-card:hover .offer-overlay {
  opacity: 1;
  transform: translateY(0);
}

.discount-pill {
  position: absolute;
  top: -2rem;
  right: 1rem;
  background: linear-gradient(135deg, var(--primary-color), var(--accent-color));
  color: white;
  padding: 0.5rem 1rem;
  border-radius: 2rem;
  font-weight: bold;
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
}

.price-container {
  display: flex;
  align-items: center;
  gap: 1rem;
  justify-content: center;
  margin-top: 0.5rem;
}

.original-price {
  color: #cbd5e1;
  text-decoration: line-through;
  font-size: 1.1rem;
}

.final-price {
  color: white;
  font-size: 1.5rem;
  font-weight: bold;
}

.custom-carousel :deep(.p-carousel-indicators) {
  padding: 1rem 0;
}

.custom-carousel :deep(.p-carousel-indicator button) {
  width: 1rem;
  height: 1rem;
  border-radius: 50%;
  background: var(--primary-color);
  opacity: 0.3;
  transition: all 0.3s ease;
}

.custom-carousel :deep(.p-carousel-indicator.p-highlight button) {
  opacity: 1;
  transform: scale(1.2);
}
</style>