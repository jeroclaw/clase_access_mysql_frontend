<template>
  <aside class="sidebar">
    <div class="sidebar-header">
      <h2>Panel de Control</h2>
    </div>

    <nav class="sidebar-nav">
      <button 
        :class="['nav-btn', { active: recursoActivo === 'clientes' }]" 
        @click="seleccionarRecurso('clientes')"
      >
        <span class="icon">👤</span> <span>Clientes</span>
      </button>
      
      <button 
        :class="['nav-btn', { active: recursoActivo === 'productos' }]" 
        @click="seleccionarRecurso('productos')"
      >
        <span class="icon">📦</span> <span>Productos</span>
      </button>
      
      <button 
        :class="['nav-btn', { active: recursoActivo === 'ordenes' }]" 
        @click="seleccionarRecurso('ordenes')"
      >
        <span class="icon">🛒</span> <span>Órdenes</span>
      </button>
      
      <button 
        :class="['nav-btn', { active: recursoActivo === 'detalle-ordenes' }]" 
        @click="seleccionarRecurso('detalle-ordenes')"
      >
        <span class="icon">📋</span> <span>Detalles</span>
      </button>
    </nav>
  </aside>
</template>

<script setup>
import { ref, defineEmits } from 'vue';
const emit = defineEmits(['cambiarRecurso']);

// Estado local para saber qué botón debe verse "activo" [cite: 61, 205]
const recursoActivo = ref('clientes');

const seleccionarRecurso = (nuevoRecurso) => {
  recursoActivo.value = nuevoRecurso; // Actualiza el estilo visual
  emit('cambiarRecurso', nuevoRecurso); // Notifica al componente padre
};
</script>

<style scoped>
.sidebar {
  width: 260px;
  background-color: #111827; /* Gris muy oscuro, casi negro */
  color: #f9fafb;
  display: flex;
  flex-direction: column;
  box-shadow: 4px 0 10px rgba(0, 0, 0, 0.1);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  z-index: 10;
}

.sidebar-header {
  padding: 24px 20px;
  border-bottom: 1px solid #374151; /* Borde separador sutil */
  margin-bottom: 16px;
}

.sidebar-header h2 {
  margin: 0;
  font-size: 1.4rem;
  font-weight: 600;
  color: #ffffff;
  letter-spacing: 0.05em;
}

.sidebar-nav {
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 0 16px;
}

.nav-btn {
  display: flex;
  align-items: center;
  gap: 14px;
  width: 100%;
  padding: 12px 16px;
  background: transparent;
  border: none;
  border-radius: 8px;
  color: #d1d5db;
  font-size: 1rem;
  font-weight: 500;
  text-align: left;
  cursor: pointer;
  transition: all 0.2s ease;
}

.nav-btn:hover {
  background-color: #1f2937; /* Gris un poco más claro al pasar el mouse */
  color: #ffffff;
}

/* Estilo para el botón seleccionado */
.nav-btn.active {
  background-color: #4f46e5; /* Color primario (Índigo) que hace juego con tu GenericManager */
  color: #ffffff;
  box-shadow: 0 4px 6px -1px rgba(79, 70, 229, 0.4);
}

.icon {
  font-size: 1.25rem;
}
</style>