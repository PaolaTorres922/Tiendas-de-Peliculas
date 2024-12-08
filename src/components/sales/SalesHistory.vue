<template>
  <div class="sales-history">
    <DataTable 
      :value="sales" 
      :paginator="true" 
      :rows="10"
      :filters="filters"
      filterDisplay="menu"
      :globalFilterFields="['movieTitle', 'date']"
      v-model:filters="filters"
      class="p-datatable-lg"
      responsiveLayout="stack"
      :rowHover="true"
      stripedRows
    >
      <template #header>
        <div class="flex flex-column sm:flex-row sm:justify-content-between sm:align-items-center">
          <Button 
            type="button" 
            icon="pi pi-filter-slash" 
            label="Limpiar Filtros" 
            @click="clearFilter"
            class="mb-2 sm:mb-0"
          />
          <span class="p-input-icon-left">
            <i class="pi pi-search" />
            <InputText 
              v-model="filters['global'].value" 
              placeholder="Buscar..." 
              class="w-full sm:w-auto"
            />
          </span>
        </div>
      </template>

      <Column field="date" header="Fecha" sortable>
        <template #body="slotProps">
          {{ formatDate(slotProps.data.date) }}
        </template>
      </Column>
      
      <Column field="movieTitle" header="Película" sortable></Column>
      
      <Column field="quantity" header="Cantidad" sortable>
        <template #body="slotProps">
          <Badge :value="slotProps.data.quantity" severity="info" />
        </template>
      </Column>
      
      <Column field="originalPrice" header="Precio Original" sortable>
        <template #body="slotProps">
          <span class="text-primary font-semibold">
            ${{ slotProps.data.originalPrice.toFixed(2) }}
          </span>
        </template>
      </Column>
      
      <Column field="discount" header="Descuento" sortable>
        <template #body="slotProps">
          <Tag 
            :value="`${slotProps.data.discount}%`"
            :severity="getDiscountSeverity(slotProps.data.discount)"
          />
        </template>
      </Column>
      
      <Column field="finalPrice" header="Precio Final" sortable>
        <template #body="slotProps">
          <span class="text-green-600 font-bold">
            ${{ slotProps.data.finalPrice.toFixed(2) }}
          </span>
        </template>
      </Column>
    </DataTable>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { FilterMatchMode } from 'primevue/api';
import DataTable from 'primevue/datatable';
import Column from 'primevue/column';
import Button from 'primevue/button';
import InputText from 'primevue/inputtext';
import Badge from 'primevue/badge';
import Tag from 'primevue/tag';

const props = defineProps({
  sales: {
    type: Array,
    required: true
  }
});

const filters = ref({
  global: { value: null, matchMode: FilterMatchMode.CONTAINS }
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

const getDiscountSeverity = (discount) => {
  if (discount >= 50) return 'danger';
  if (discount >= 25) return 'warning';
  return 'success';
};

const clearFilter = () => {
  filters.value = {
    global: { value: null, matchMode: FilterMatchMode.CONTAINS }
  };
};
</script>

<style scoped>
.sales-history {
  background: rgba(255, 255, 255, 0.9);
  border-radius: 1rem;
  padding: 1.5rem;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

:deep(.p-datatable) {
  border-radius: 0.5rem;
  overflow: hidden;
}

:deep(.p-datatable .p-datatable-header) {
  background: transparent;
  border: none;
  padding: 1rem;
}

:deep(.p-datatable .p-datatable-thead > tr > th) {
  background: var(--primary-color);
  color: white;
  padding: 1rem;
  font-weight: 600;
}

:deep(.p-datatable .p-datatable-tbody > tr) {
  transition: background-color 0.3s ease;
}

:deep(.p-datatable .p-datatable-tbody > tr:hover) {
  background: rgba(99, 102, 241, 0.1);
}

@media screen and (max-width: 768px) {
  .sales-history {
    padding: 1rem;
  }
  
  :deep(.p-datatable .p-datatable-thead > tr > th) {
    padding: 0.5rem;
  }
}
</style>