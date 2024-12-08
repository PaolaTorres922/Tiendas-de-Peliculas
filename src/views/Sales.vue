<template>
  <div class="sales-dashboard">
    <TabView>
      <!-- Resumen de Ventas -->
      <TabPanel header="Resumen">
        <div class="grid">
          <div class="col-12 md:col-4">
            <SalesCard 
              title="Total de Ventas" 
              :value="`$${totalSalesAmount}`"
              icon="pi pi-money-bill"
            />
          </div>
          <div class="col-12 md:col-4">
            <SalesCard 
              title="Películas Vendidas" 
              :value="totalMoviesSold"
              icon="pi pi-shopping-cart"
            />
          </div>
          <div class="col-12 md:col-4">
            <SalesCard 
              title="Descuentos Aplicados" 
              :value="`$${totalDiscounts}`"
              icon="pi pi-percentage"
            />
          </div>
        </div>

        <div class="grid mt-4">
          <div class="col-12 lg:col-6">
            <SalesChart 
              title="Ventas Mensuales"
              type="bar"
              :data="monthlySalesData"
            />
          </div>
          <div class="col-12 lg:col-6">
            <SalesChart 
              title="Películas Más Vendidas"
              type="pie"
              :data="topMoviesData"
            />
          </div>
        </div>
      </TabPanel>

      <!-- Historial de Ventas -->
      <TabPanel header="Historial">
        <SalesHistory :sales="salesWithDetails" />
      </TabPanel>
    </TabView>
  </div>
</template>

<script setup>
import { computed } from 'vue';
import { useMovieStore } from '../stores/movies';
import TabView from 'primevue/tabview';
import TabPanel from 'primevue/tabpanel';
import SalesCard from '../components/sales/SalesCard.vue';
import SalesChart from '../components/sales/SalesChart.vue';
import SalesHistory from '../components/sales/SalesHistory.vue';

const movieStore = useMovieStore();

const salesWithDetails = computed(() => {
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

const totalSalesAmount = computed(() => {
  return salesWithDetails.value.reduce((total, sale) => total + sale.finalPrice, 0).toFixed(2);
});

const totalMoviesSold = computed(() => {
  return salesWithDetails.value.reduce((total, sale) => total + sale.quantity, 0);
});

const totalDiscounts = computed(() => {
  return salesWithDetails.value.reduce((total, sale) => {
    const discount = (sale.originalPrice - sale.finalPrice);
    return total + discount;
  }, 0).toFixed(2);
});

const monthlySalesData = computed(() => {
  const monthlyData = {};
  salesWithDetails.value.forEach(sale => {
    const month = sale.date.toLocaleString('es-ES', { month: 'long' });
    monthlyData[month] = (monthlyData[month] || 0) + sale.finalPrice;
  });

  return {
    labels: Object.keys(monthlyData),
    datasets: [{
      label: 'Ventas Mensuales',
      data: Object.values(monthlyData),
      backgroundColor: '#6366f1',
      borderColor: '#4f46e5',
      borderWidth: 1
    }]
  };
});

const topMoviesData = computed(() => {
  const movieSales = {};
  salesWithDetails.value.forEach(sale => {
    movieSales[sale.movieTitle] = (movieSales[sale.movieTitle] || 0) + sale.quantity;
  });

  return {
    labels: Object.keys(movieSales),
    datasets: [{
      data: Object.values(movieSales),
      backgroundColor: [
        '#6366f1',
        '#8b5cf6',
        '#ec4899',
        '#f43f5e',
        '#f97316'
      ],
      borderWidth: 1
    }]
  };
});
</script>

<style scoped>
.sales-dashboard {
  padding: 1rem;
  background: linear-gradient(135deg, var(--surface-ground) 0%, var(--surface-section) 100%);
  min-height: calc(100vh - 4rem);
}

:deep(.p-tabview-panels) {
  background: transparent;
  padding: 1.5rem 0;
}

:deep(.p-tabview-nav) {
  border: none;
  background: transparent;
}

:deep(.p-tabview-nav-link) {
  background: rgba(255, 255, 255, 0.9) !important;
  border: 1px solid rgba(255, 255, 255, 0.2) !important;
  border-radius: 0.5rem !important;
  margin-right: 0.5rem !important;
  transition: all 0.3s ease !important;
}

:deep(.p-tabview-selected .p-tabview-nav-link) {
  background: var(--primary-color) !important;
  color: white !important;
  border-color: var(--primary-color) !important;
}
</style>