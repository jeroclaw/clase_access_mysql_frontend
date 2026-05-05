<template>
  <aside class="sidebar">
    <div class="sidebar-header">
      <h2>Panel de Control</h2>
    </div>

    <nav class="sidebar-nav">
      <button 
        :class="['nav-btn', { active: recursoActivo === 'inicio' }]" 
        @click="seleccionarRecurso('inicio')"
      >
        <span class="icon">🏠</span> <span>Inicio</span>
      </button>

      <button 
        v-if="hasPermiso('ver_clientes')"
        :class="['nav-btn', { active: recursoActivo === 'clientes' }]" 
        @click="seleccionarRecurso('clientes')"
      >
        <span class="icon">👤</span> <span>Clientes</span>
      </button>
      
      <button 
        v-if="hasPermiso('ver_productos')"
        :class="['nav-btn', { active: recursoActivo === 'productos' }]" 
        @click="seleccionarRecurso('productos')"
      >
        <span class="icon">📦</span> <span>Productos</span>
      </button>
      
      <button 
        v-if="hasPermiso('ver_ordenes')"
        :class="['nav-btn', { active: recursoActivo === 'ordenes' }]" 
        @click="seleccionarRecurso('ordenes')"
      >
        <span class="icon">🛒</span> <span>Órdenes</span>
      </button>
      
      <button 
        v-if="hasPermiso('ver_detalle_ordenes')"
        :class="['nav-btn', { active: recursoActivo === 'detalle-ordenes' }]" 
        @click="seleccionarRecurso('detalle-ordenes')"
      >
        <span class="icon">📋</span> <span>Detalles</span>
      </button>
      <button 
        v-if="hasPermiso('ver_envios')"
        :class="['nav-btn', { active: recursoActivo === 'envios' }]" 
        @click="seleccionarRecurso('envios')"
      >
        <span class="icon">📋</span> <span>Envios</span>
      </button>

      <hr class="sidebar-divider" v-if="hasAnyAdminPerm" />

      <button 
        v-if="hasPermiso('ver_usuarios')"
        :class="['nav-btn', { active: recursoActivo === 'users' }]" 
        @click="seleccionarRecurso('users')"
      >
        <span class="icon">👥</span> <span>Usuarios</span>
      </button>
      <button 
        v-if="hasPermiso('ver_roles')"
        :class="['nav-btn', { active: recursoActivo === 'roles' }]" 
        @click="seleccionarRecurso('roles')"
      >
        <span class="icon">🛡️</span> <span>Roles</span>
      </button>
      <button 
        v-if="hasPermiso('ver_permisos')"
        :class="['nav-btn', { active: recursoActivo === 'permissions' }]" 
        @click="seleccionarRecurso('permissions')"
      >
        <span class="icon">🔑</span> <span>Permisos</span>
      </button>
      <button 
        v-if="hasPermiso('ver_seguridad')"
        :class="['nav-btn', { active: recursoActivo === 'seguridad' }]" 
        @click="seleccionarRecurso('seguridad')"
      >
        <span class="icon">🔐</span> <span>Seguridad</span>
      </button>

      <hr class="sidebar-divider" />

      <button 
        class="nav-btn logout-btn" 
        @click="cerrarSesion"
      >
        <span class="icon">🚪</span> <span>Cerrar sesión</span>
      </button>
    </nav>
  </aside>
</template>

<script setup>
import { ref, defineEmits, computed } from 'vue';
const emit = defineEmits(['cambiarRecurso', 'logout']);

// Estado local para saber qué botón debe verse "activo" [cite: 61, 205]
const recursoActivo = ref('inicio');

const hasPermiso = (permiso) => {
  try {
    const roles = JSON.parse(localStorage.getItem('roles') || '[]');
    // Si es admin, ignoramos los permisos y devolvemos true directamente
    if (roles.includes('Admin') || roles.includes('Administrador')) return true;
  } catch (e) {
    // Ignoramos el error si no hay roles válidos en el localStorage
  }

  try {
    console.log('permisos', localStorage.getItem('permisos'));
    const permisosGuardados = JSON.parse(localStorage.getItem('permisos') || '[]');
    return permisosGuardados.includes(permiso);
  } catch {
    return false;
  }
};

const hasAnyAdminPerm = computed(() => {
  return hasPermiso('ver_usuarios') || hasPermiso('ver_roles') || hasPermiso('ver_permisos') || hasPermiso('ver_seguridad');
});

const seleccionarRecurso = (nuevoRecurso) => {
  recursoActivo.value = nuevoRecurso; // Actualiza el estilo visual
  emit('cambiarRecurso', nuevoRecurso); // Notifica al componente padre
};

const cerrarSesion = () => {
  localStorage.removeItem('token'); // Borramos el token de acceso
  localStorage.removeItem('permisos'); // Borramos los permisos al salir
  localStorage.removeItem('roles'); // Borramos los roles al salir
  emit('logout'); // Le avisamos a App.vue que la sesión se cerró
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

.sidebar-divider {
  border: none;
  border-top: 1px solid #374151;
  margin: 10px 0;
}

.logout-btn {
  color: #fca5a5; /* Letra rojiza suave para destacar */
  margin-top: auto;
}
.logout-btn:hover {
  background-color: #7f1d1d; /* Fondo rojo oscuro al pasar el mouse */
  color: #ffffff;
}
</style>