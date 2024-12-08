<template>
  <div class="movie-card">
    <Card class="h-full">
      <template #header>
        <div class="movie-image">
          <img :src="movie.image" :alt="movie.title" class="w-full h-15rem object-cover" />
          <span v-if="movie.discount > 0" class="discount-badge">
            -{{ movie.discount }}% OFF
          </span>
          <span :class="['stock-badge', getStockSeverity(movie.stock)]">
            {{ getStockLabel(movie.stock) }}
          </span>
        </div>
      </template>
      
      <template #title>
        <div class="text-2xl font-bold mb-2">{{ movie.title }}</div>
      </template>
      
      <template #subtitle>
        <div class="flex align-items-center gap-3 mb-3">
          <span class="text-xl" :class="{ 'line-through text-500': movie.discount > 0 }">
            ${{ movie.price }}
          </span>
          <span v-if="movie.discount > 0" class="text-xl text-primary font-bold">
            ${{ calculateDiscountedPrice(movie) }}
          </span>
        </div>
      </template>
      
      <template #content>
        <p class="text-700 mb-4">{{ movie.description }}</p>
        <div class="flex align-items-center gap-2">
          <i class="pi pi-star-fill text-yellow-500"></i>
          <span class="font-semibold">{{ movie.rating }}/5</span>
        </div>
      </template>
      
      <template #footer>
        <div class="flex flex-column gap-2">
          <Button v-if="!isAuthenticated" 
                 label="Iniciar sesión para comprar" 
                 icon="pi pi-sign-in" 
                 @click="$router.push('/login')" 
                 class="p-button-outlined" />
          <Button v-else 
                 :label="movie.stock > 0 ? 'Comprar ahora' : 'Sin Stock'" 
                 :icon="movie.stock > 0 ? 'pi pi-shopping-cart' : 'pi pi-times-circle'" 
                 :disabled="movie.stock === 0"
                 @click="$emit('purchase', movie)" 
                 :class="{'p-button-danger': movie.stock === 0}" />
        </div>
      </template>
    </Card>
  </div>
</template>

<script setup>
import { computed } from 'vue';
import Card from 'primevue/card';
import Button from 'primevue/button';

const props = defineProps({
  movie: {
    type: Object,
    required: true
  },
  isAuthenticated: {
    type: Boolean,
    default: false
  }
});

defineEmits(['purchase']);

const getStockSeverity = (stock) => {
  if (stock === 0) return 'danger';
  if (stock < 3) return 'warning';
  return 'success';
};

const getStockLabel = (stock) => {
  if (stock === 0) return 'Sin Stock';
  if (stock < 3) return `¡Solo ${stock} disponibles!`;
  return `${stock} en stock`;
};

const calculateDiscountedPrice = (movie) => {
  return (movie.price * (1 - movie.discount / 100)).toFixed(2);
};
</script>