<template>
  <div id="app" class="dashboard-layout">
    <SidebarNavigation @cambiarRecurso="actualizarVista" />

    <main class="content">
      <GenericManager :recursoActive="recursoActual" />
    </main>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import SidebarNavigation from './components/SidebarNavigation.vue';
import GenericManager from './components/GenericManager.vue';

// Estado inicial: por defecto carga 'clientes' [cite: 61, 205]
const recursoActual = ref('clientes');

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
</style>
