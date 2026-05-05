<template>
  <div class="login-wrapper">
    <form @submit.prevent="handleLogin">
      <h2>Iniciar sesión</h2>

      <input
        v-model="form.email"
        type="email"
        placeholder="Email"
        required
      />
      <input
        v-model="form.password"
        type="password"
        placeholder="Contraseña"
        required
      />

      <button type="submit" :disabled="loading">
        {{ loading ? 'Ingresando...' : 'Ingresar' }}
      </button>

      <p v-if="error" class="error-msg">{{ error }}</p>
    </form>
  </div>
</template>

<script setup>
import { ref, defineEmits } from 'vue'
import axios from 'axios'

const emit      = defineEmits(['loginSuccess'])
const loading   = ref(false)
const error     = ref('')
const form      = ref({ email: '', password: '' })


async function handleLogin() {
  const baseURL = 'http://localhost:8081/api';
  loading.value = true
  error.value   = ''
  try {
    const { data } = await axios.post(`${baseURL}/login`, form.value)

    console.log(data.data.token)
    // Guardar token y configurar cabecera global
    localStorage.setItem('token', data.data.token)
    axios.defaults.headers.common['Authorization'] = `Bearer ${data.data.token}`

    // Guardar los permisos en localStorage (Ajustar si tu API los devuelve en otra ruta como data.data.user.permisos)
    const permisosUsuario = data.data.permissions || []; 
    localStorage.setItem('permisos', JSON.stringify(permisosUsuario));

    // Guardar los roles en localStorage para validaciones de administrador
    const rolesUsuario = data.data.roles || [];
    localStorage.setItem('roles', JSON.stringify(rolesUsuario));

    // Le avisamos a App.vue que el login fue correcto
    emit('loginSuccess')
  } catch (e) {
    error.value = e.response?.data?.message ?? 'Error de conexión'
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.login-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #f9fafb;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}
form {
  background: #ffffff;
  padding: 40px;
  border-radius: 12px;
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
  display: flex;
  flex-direction: column;
  gap: 16px;
  width: 100%;
  max-width: 360px;
}
h2 {
  margin: 0 0 10px;
  color: #111827;
  text-align: center;
}
input {
  padding: 12px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 1rem;
}
button {
  padding: 12px;
  background-color: #4f46e5;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.2s;
}
button:hover:not(:disabled) {
  background-color: #4338ca;
}
button:disabled {
  background-color: #9ca3af;
  cursor: not-allowed;
}
.error-msg {
  color: #dc2626;
  font-size: 0.875rem;
  text-align: center;
  margin: 0;
}
</style>