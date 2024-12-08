<template>
  <div class="admin-dashboard">
    <TabView>
      <TabPanel header="Gestión de Películas">
        <Toolbar class="mb-4">
          <template #start>
            <h2 class="m-0">Catálogo de Películas</h2>
          </template>
          <template #end>
            <Button label="Nueva Película" icon="pi pi-plus" @click="openNew" />
          </template>
        </Toolbar>

        <DataTable 
          :value="movies" 
          :paginator="true" 
          :rows="10"
          responsiveLayout="stack"
          class="p-datatable-lg"
          :rowHover="true"
          v-model:selection="selectedMovies"
          dataKey="id"
          :filters="filters"
          filterDisplay="menu"
        >
          <template #header>
            <div class="flex justify-content-between">
              <Button 
                label="Eliminar Seleccionados" 
                icon="pi pi-trash" 
                severity="danger" 
                @click="confirmDeleteSelected" 
                :disabled="!selectedMovies || !selectedMovies.length"
              />
              <span class="p-input-icon-left">
                <i class="pi pi-search" />
                <InputText v-model="filters['global'].value" placeholder="Buscar película..." />
              </span>
            </div>
          </template>

          <Column selectionMode="multiple" headerStyle="width: 3rem"></Column>

          <Column field="image" header="Imagen">
            <template #body="slotProps">
              <img :src="slotProps.data.image" :alt="slotProps.data.title" class="movie-thumbnail shadow-2" />
            </template>
          </Column>

          <Column field="title" header="Título" sortable></Column>
          
          <Column field="price" header="Precio" sortable>
            <template #body="slotProps">
              <span class="text-primary font-bold">${{ slotProps.data.price }}</span>
            </template>
          </Column>
          
          <Column field="stock" header="Stock" sortable>
            <template #body="slotProps">
              <Tag :value="slotProps.data.stock" 
                   :severity="getStockSeverity(slotProps.data.stock)" />
            </template>
          </Column>
          
          <Column field="discount" header="Descuento" sortable>
            <template #body="slotProps">
              <Tag v-if="slotProps.data.discount > 0" 
                   :value="`${slotProps.data.discount}% OFF`"
                   severity="danger" />
              <span v-else>-</span>
            </template>
          </Column>

          <Column header="Acciones">
            <template #body="slotProps">
              <div class="flex gap-2">
                <Button 
                  icon="pi pi-pencil" 
                  class="p-button-rounded p-button-success" 
                  @click="editMovie(slotProps.data)" 
                />
                <Button 
                  icon="pi pi-trash" 
                  class="p-button-rounded p-button-danger" 
                  @click="confirmDelete(slotProps.data)" 
                />
              </div>
            </template>
          </Column>
        </DataTable>
      </TabPanel>
    </TabView>

    <!-- Diálogo de Edición/Creación -->
    <Dialog 
      v-model:visible="movieDialog" 
      :header="dialogTitle" 
      modal 
      class="p-fluid"
      :style="{width: '50vw'}"
    >
      <div class="grid formgrid">
        <div class="field col-12">
          <label for="title">Título</label>
          <InputText id="title" v-model="movie.title" required autofocus />
        </div>

        <div class="field col-12 md:col-6">
          <label for="price">Precio</label>
          <InputNumber id="price" v-model="movie.price" mode="currency" currency="USD" />
        </div>

        <div class="field col-12 md:col-6">
          <label for="stock">Stock</label>
          <InputNumber id="stock" v-model="movie.stock" />
        </div>

        <div class="field col-12 md:col-6">
          <label for="discount">Descuento (%)</label>
          <InputNumber id="discount" v-model="movie.discount" suffix="%" :min="0" :max="100" />
        </div>

        <div class="field col-12">
          <label for="image">URL de la Imagen</label>
          <InputText id="image" v-model="movie.image" />
          <small class="text-primary">Ingrese la URL de la imagen de la película</small>
        </div>

        <div class="field col-12">
          <label for="description">Descripción</label>
          <Textarea id="description" v-model="movie.description" rows="3" autoResize />
        </div>
      </div>

      <template #footer>
        <Button label="Cancelar" icon="pi pi-times" class="p-button-text" @click="hideDialog" />
        <Button label="Guardar" icon="pi pi-check" @click="saveMovie" />
      </template>
    </Dialog>

    <!-- Diálogo de Confirmación de Eliminación -->
    <Dialog 
      v-model:visible="deleteMovieDialog" 
      header="Confirmar" 
      modal 
      :style="{width: '450px'}"
    >
      <div class="confirmation-content">
        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
        <span>¿Estás seguro de que quieres eliminar {{ movie.title }}?</span>
      </div>
      <template #footer>
        <Button label="No" icon="pi pi-times" class="p-button-text" @click="deleteMovieDialog = false" />
        <Button label="Sí" icon="pi pi-check" class="p-button-danger" @click="deleteMovie" />
      </template>
    </Dialog>

    <!-- Diálogo de Confirmación de Eliminación Múltiple -->
    <Dialog 
      v-model:visible="deleteMoviesDialog" 
      header="Confirmar" 
      modal 
      :style="{width: '450px'}"
    >
      <div class="confirmation-content">
        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
        <span>¿Estás seguro de que quieres eliminar las películas seleccionadas?</span>
      </div>
      <template #footer>
        <Button label="No" icon="pi pi-times" class="p-button-text" @click="deleteMoviesDialog = false" />
        <Button label="Sí" icon="pi pi-check" class="p-button-danger" @click="deleteSelectedMovies" />
      </template>
    </Dialog>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import { useMovieStore } from '../stores/movies';
import { useToast } from 'primevue/usetoast';
import { FilterMatchMode } from 'primevue/api';
import DataTable from 'primevue/datatable';
import Column from 'primevue/column';
import Button from 'primevue/button';
import Dialog from 'primevue/dialog';
import InputText from 'primevue/inputtext';
import InputNumber from 'primevue/inputnumber';
import Textarea from 'primevue/textarea';
import Toolbar from 'primevue/toolbar';
import TabView from 'primevue/tabview';
import TabPanel from 'primevue/tabpanel';
import Tag from 'primevue/tag';

const movieStore = useMovieStore();
const toast = useToast();

const movies = computed(() => movieStore.movies);
const movieDialog = ref(false);
const deleteMovieDialog = ref(false);
const deleteMoviesDialog = ref(false);
const movie = ref({});
const selectedMovies = ref();
const filters = ref({
  global: { value: null, matchMode: FilterMatchMode.CONTAINS }
});

const dialogTitle = computed(() => movie.value.id ? 'Editar Película' : 'Nueva Película');

const getStockSeverity = (stock) => {
  if (stock === 0) return 'danger';
  if (stock < 3) return 'warning';
  return 'success';
};

const openNew = () => {
  movie.value = {
    title: '',
    price: 0,
    stock: 0,
    discount: 0,
    description: '',
    image: ''
  };
  movieDialog.value = true;
};

const hideDialog = () => {
  movieDialog.value = false;
};

const saveMovie = () => {
  if (movie.value.title.trim()) {
    if (movie.value.id) {
      movieStore.updateMovie(movie.value.id, movie.value);
      toast.add({
        severity: 'success',
        summary: 'Éxito',
        detail: 'Película actualizada',
        life: 3000
      });
    } else {
      movieStore.addMovie(movie.value);
      toast.add({
        severity: 'success',
        summary: 'Éxito',
        detail: 'Película creada',
        life: 3000
      });
    }
    movieDialog.value = false;
  }
};

const editMovie = (data) => {
  movie.value = { ...data };
  movieDialog.value = true;
};

const confirmDelete = (data) => {
  movie.value = data;
  deleteMovieDialog.value = true;
};

const deleteMovie = () => {
  movieStore.deleteMovie(movie.value.id);
  deleteMovieDialog.value = false;
  toast.add({
    severity: 'success',
    summary: 'Éxito',
    detail: 'Película eliminada',
    life: 3000
  });
};

const confirmDeleteSelected = () => {
  deleteMoviesDialog.value = true;
};

const deleteSelectedMovies = () => {
  selectedMovies.value.forEach(movie => {
    movieStore.deleteMovie(movie.id);
  });
  deleteMoviesDialog.value = false;
  selectedMovies.value = null;
  toast.add({
    severity: 'success',
    summary: 'Éxito',
    detail: 'Películas eliminadas',
    life: 3000
  });
};
</script>

<style scoped>
.admin-dashboard {
  padding: 1rem;
}

.movie-thumbnail {
  width: 100px;
  height: 150px;
  object-fit: cover;
  border-radius: 0.5rem;
}

:deep(.p-dialog-content) {
  padding: 2rem;
}

.confirmation-content {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1rem;
}

:deep(.p-toolbar) {
  background: transparent;
  border: none;
  padding: 1rem 0;
}

:deep(.p-datatable) {
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(10px);
  border-radius: 1rem;
  overflow: hidden;
}

@media screen and (max-width: 768px) {
  .movie-thumbnail {
    width: 80px;
    height: 120px;
  }
}
</style>