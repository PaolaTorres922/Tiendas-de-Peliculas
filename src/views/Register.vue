<template>
  <div class="flex align-items-center justify-content-center min-h-screen">
    <div class="surface-card p-4 shadow-2 border-round w-full lg:w-4">
      <h2 class="text-center text-2xl mb-4">Registro</h2>
      <form @submit.prevent="handleRegister" class="flex flex-column gap-3">
        <span class="p-float-label">
          <InputText id="name" v-model="name" class="w-full" required />
          <label for="name">Nombre completo</label>
        </span>
        
        <span class="p-float-label">
          <InputText 
            id="email" 
            v-model="email" 
            type="email" 
            class="w-full" 
            required 
          />
          <label for="email">Correo electrónico</label>
        </span>
        
        <span class="p-float-label">
          <Password 
            v-model="password" 
            :feedback="true"
            class="w-full"
            toggleMask
            required
          />
          <label for="password">Contraseña</label>
        </span>

        <div class="field-radiobutton">
          <h3 class="mb-2">Tipo de Usuario:</h3>
          <div class="flex flex-column gap-2">
            <div class="flex align-items-center">
              <RadioButton 
                v-model="userType" 
                inputId="cliente" 
                name="userType" 
                value="client" 
              />
              <label for="cliente" class="ml-2">Cliente</label>
            </div>
            <div class="flex align-items-center">
              <RadioButton 
                v-model="userType" 
                inputId="admin" 
                name="userType" 
                value="admin" 
              />
              <label for="admin" class="ml-2">Administrador</label>
            </div>
          </div>
        </div>
        
        <Button type="submit" label="Registrarse" class="w-full mt-2" />
        
        <div class="text-center mt-3">
          ¿Ya tienes cuenta? 
          <router-link to="/login" class="font-bold text-primary">Iniciar sesión</router-link>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import { useAuthStore } from '../stores/auth';
import { useToast } from 'primevue/usetoast';
import InputText from 'primevue/inputtext';
import Password from 'primevue/password';
import Button from 'primevue/button';
import RadioButton from 'primevue/radiobutton';

const router = useRouter();
const authStore = useAuthStore();
const toast = useToast();

const name = ref('');
const email = ref('');
const password = ref('');
const userType = ref('client');

const handleRegister = () => {
  if (authStore.register({ 
    name: name.value, 
    email: email.value, 
    password: password.value,
    role: userType.value
  })) {
    toast.add({
      severity: 'success',
      summary: '¡Registro exitoso!',
      detail: `Bienvenido a CineStore como ${userType.value === 'admin' ? 'Administrador' : 'Cliente'}`,
      life: 3000
    });
    router.push('/');
  } else {
    toast.add({
      severity: 'error',
      summary: 'Error',
      detail: 'El correo electrónico ya está registrado',
      life: 3000
    });
  }
};
</script>

<style scoped>
.field-radiobutton {
  background: var(--surface-card);
  padding: 1rem;
  border-radius: 0.5rem;
  border: 1px solid var(--surface-border);
}

:deep(.p-password-input) {
  width: 100%;
}

:deep(.p-radiobutton .p-radiobutton-box.p-highlight) {
  border-color: var(--primary-color);
  background: var(--primary-color);
}
</style>