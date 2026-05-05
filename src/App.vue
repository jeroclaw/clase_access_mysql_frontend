<template>
  <div id="app">
    <!-- Mostrar pantalla de Login si estamos en su ruta o no hay sesión -->
    <LoginView v-if="mostrarLogin" @loginSuccess="manejarLoginExitoso" />

    <!-- Mostrar Layout principal si ya inició sesión -->
    <div v-else class="dashboard-layout">
      <SidebarNavigation @cambiarRecurso="actualizarVista" @logout="manejarLogout" />

      <main class="content">
        <div v-if="recursoActual === 'inicio'" class="welcome-container">
          <h1>Bienvenido al sistema del curso</h1>
          <p>Seleccioná una opción del menú lateral para comenzar a trabajar.</p>
        </div>
        <GenericManager v-else-if="recursoActual !== 'seguridad'" :recursoActive="recursoActual" />
        <SeguridadManager v-else />
      </main>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';
import SidebarNavigation from './components/SidebarNavigation.vue';
import GenericManager from './components/GenericManager.vue';
import SeguridadManager from './components/SeguridadManager.vue';
import LoginView from './components/LoginView.vue';

// 1. Inyectar el token en la configuración global por defecto al iniciar
const tokenGuardado = localStorage.getItem('token');
if (tokenGuardado && tokenGuardado !== 'undefined') {
  axios.defaults.headers.common['Authorization'] = `Bearer ${tokenGuardado}`;
}

// Configurar Axios para que envíe el token automáticamente en cada petición
axios.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if (token && token !== 'undefined') {
    // 2. Asegurarnos de que el objeto headers exista
    config.headers = config.headers || {};
    // 3. Asignar el token de forma segura
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
}, error => {
  return Promise.reject(error);
});

// Estado para controlar si mostramos el login (si no hay token guardado)
const mostrarLogin = ref(!localStorage.getItem('token'));

const manejarLoginExitoso = () => {
  mostrarLogin.value = false; // Oculta el login y muestra el panel
};

const manejarLogout = () => {
  // Vuelve a mostrar la pantalla de Login
  mostrarLogin.value = true;
};

// Estado inicial: por defecto carga la pantalla de inicio
const recursoActual = ref('inicio');

// Función que se ejecuta cuando hacés clic en el menú [cite: 116]
const actualizarVista = (nuevoRecurso) => {
  recursoActual.value = nuevoRecurso;
};
</script>

<style>
.dashboard-layout {
  display: flex; /* Para que el menú quede a la izquierda y el contenido a la derecha */
  min-height: 100vh;
}
.content {
  flex-grow: 1;
  padding: 20px;
}

.welcome-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 80vh;
  text-align: center;
  color: #374151;
}
.welcome-container h1 {
  font-size: 2.5rem;
  margin-bottom: 10px;
  color: #111827;
}
.welcome-container p {
  font-size: 1.1rem;
  color: #6b7280;
}
</style>
