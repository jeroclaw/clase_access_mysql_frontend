<template>
  <div class="manager-container">
    <div class="header">
      <div class="header-icon">🔐</div>
      <div>
        <h1 class="title">Gestión de Seguridad</h1>
        <p class="subtitle">Administrá roles, permisos y usuarios</p>
      </div>
    </div>

    <div v-if="cargando" class="loading">
      <div class="spinner"></div>
      <span>Cargando datos de seguridad...</span>
    </div>

    <div v-if="mensaje.texto" :class="['alerta', `alerta-${mensaje.tipo}`]">
      {{ mensaje.texto }}
    </div>

    <div class="security-grid" v-if="!cargando">

      <!-- PANEL 1: Roles -> Permisos -->
      <div class="card" v-if="configRecurso.canRolesPermisos">
        <div class="card-header">
          <span class="card-icon">🎭</span>
          <div>
            <h3 class="card-title">Permisos por Rol</h3>
            <p class="card-desc">Seleccioná un rol y asignale permisos.</p>
          </div>
        </div>

        <div class="form-group">
          <label class="form-label">Seleccionar Rol</label>
          <select v-model="rolSeleccionado" class="form-control" @change="cargarPermisosDeRol">
            <option value="" disabled>— Elegí un Rol —</option>
            <option v-for="rol in roles" :key="rol.id" :value="rol.id">
              {{ rol.name }}
            </option>
          </select>
        </div>

        <div v-if="cargandoPanel1" class="panel-loading">Cargando permisos...</div>

        <div v-if="rolSeleccionado && !cargandoPanel1" class="checkbox-grid">
          <label v-for="permiso in permisos" :key="permiso.id" class="checkbox-label">
            <input type="checkbox" :value="permiso.id" v-model="permisosDelRol" class="checkbox-input" />
            <span class="checkbox-text">{{ permiso.name }}</span>
          </label>
        </div>

        <button
          v-if="rolSeleccionado && !cargandoPanel1"
          class="btn btn-primary mt-3"
          :disabled="guardandoPanel1"
          @click="guardarPermisosRol"
        >
          {{ guardandoPanel1 ? 'Guardando...' : 'Guardar Permisos del Rol' }}
        </button>
      </div>

      <!-- PANEL 2: Usuarios -> Roles -->
      <div class="card" v-if="configRecurso.canUserRoles">
        <div class="card-header">
          <span class="card-icon">👤</span>
          <div>
            <h3 class="card-title">Roles por Usuario</h3>
            <p class="card-desc">Seleccioná un usuario y asignale roles.</p>
          </div>
        </div>

        <div class="form-group">
          <label class="form-label">Seleccionar Usuario</label>
          <select v-model="usuarioParaRol" class="form-control" @change="cargarRolesDeUsuario">
            <option value="" disabled>— Elegí un Usuario —</option>
            <option v-for="user in usuarios" :key="user.id" :value="user.id">
              {{ user.name }} ({{ user.email }})
            </option>
          </select>
        </div>

        <div v-if="cargandoPanel2" class="panel-loading">Cargando roles...</div>

        <div v-if="usuarioParaRol && !cargandoPanel2" class="checkbox-grid">
          <label v-for="rol in roles" :key="rol.id" class="checkbox-label">
            <input type="checkbox" :value="rol.id" v-model="rolesDelUsuario" class="checkbox-input" />
            <span class="checkbox-text">{{ rol.name }}</span>
          </label>
        </div>

        <button
          v-if="usuarioParaRol && !cargandoPanel2"
          class="btn btn-primary mt-3"
          :disabled="guardandoPanel2"
          @click="guardarRolesUsuario"
        >
          {{ guardandoPanel2 ? 'Guardando...' : 'Guardar Roles del Usuario' }}
        </button>
      </div>

      <!-- PANEL 3: Usuarios -> Permisos Directos -->
      <div class="card" v-if="configRecurso.canPermisosEsp">
        <div class="card-header">
          <span class="card-icon">🔑</span>
          <div>
            <h3 class="card-title">Permisos Directos</h3>
            <p class="card-desc">Asigná permisos directos a un usuario específico.</p>
          </div>
        </div>

        <div class="form-group">
          <label class="form-label">Seleccionar Usuario</label>
          <select v-model="usuarioParaPermiso" class="form-control" @change="cargarPermisosDeUsuario">
            <option value="" disabled>— Elegí un Usuario —</option>
            <option v-for="user in usuarios" :key="user.id" :value="user.id">
              {{ user.name }} ({{ user.email }})
            </option>
          </select>
        </div>

        <div v-if="cargandoPanel3" class="panel-loading">Cargando permisos...</div>

        <div v-if="usuarioParaPermiso && !cargandoPanel3" class="checkbox-grid">
          <label v-for="permiso in permisos" :key="permiso.id" class="checkbox-label">
            <input type="checkbox" :value="permiso.id" v-model="permisosDelUsuario" class="checkbox-input" />
            <span class="checkbox-text">{{ permiso.name }}</span>
          </label>
        </div>

        <button
          v-if="usuarioParaPermiso && !cargandoPanel3"
          class="btn btn-primary mt-3"
          :disabled="guardandoPanel3"
          @click="guardarPermisosUsuario"
        >
          {{ guardandoPanel3 ? 'Guardando...' : 'Guardar Permisos del Usuario' }}
        </button>
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';

// ─────────────────────────────────────────────
// CONFIGURACIÓN BASE
// ─────────────────────────────────────────────
const baseURL = 'http://localhost:8081/api';

// Si el token ya está en localStorage (ej: recarga de página), lo restauramos
const token = localStorage.getItem('token');
console.log(token);
if (token) {
  axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
}


// ─────────────────────────────────────────────
// ESTADO GLOBAL
// ─────────────────────────────────────────────
const cargando = ref(false);
const mensaje = ref({ texto: '', tipo: 'success' });

const usuarios = ref([]);
const roles = ref([]);
const permisos = ref([]);

const configRecurso = computed(() => {
  try {
    const permisos = JSON.parse(localStorage.getItem('permisos') || '[]');
    const roles = JSON.parse(localStorage.getItem('roles') || '[]');
    const isAdmin = roles.includes('Admin') || roles.includes('Administrador');
    
    return {
      canUserRoles: isAdmin || permisos.includes(`editar_userroles`),
      canRolesPermisos: isAdmin || permisos.includes(`editar_rolespermisos`),
      canPermisosEsp: isAdmin || permisos.includes(`editar_permisosesp`)
    };
  } catch {
    return { canUserRoles: false, canRolesPermisos: false, canDelete: false };
  }
});

// Panel 1
const rolSeleccionado = ref('');
const permisosDelRol = ref([]);
const cargandoPanel1 = ref(false);
const guardandoPanel1 = ref(false);

// Panel 2
const usuarioParaRol = ref('');
const rolesDelUsuario = ref([]);
const cargandoPanel2 = ref(false);
const guardandoPanel2 = ref(false);

// Panel 3
const usuarioParaPermiso = ref('');
const permisosDelUsuario = ref([]);
const cargandoPanel3 = ref(false);
const guardandoPanel3 = ref(false);

// ─────────────────────────────────────────────
// HELPERS
// ─────────────────────────────────────────────
const mostrarMensaje = (texto, tipo = 'success') => {
  mensaje.value = { texto, tipo };
  setTimeout(() => { mensaje.value = { texto: '', tipo: 'success' }; }, 4000);
};

// ─────────────────────────────────────────────
// CARGA INICIAL
// ─────────────────────────────────────────────
onMounted(async () => {
  cargando.value = true;
  try {
    const [usersRes, rolesRes, permsRes] = await Promise.all([
      axios.get(`${baseURL}/users`),
      axios.get(`${baseURL}/roles`),
      axios.get(`${baseURL}/permissions`),
    ]);
    usuarios.value = usersRes.data;
    roles.value = rolesRes.data;
    permisos.value = permsRes.data;
  } catch (error) {
    mostrarMensaje('Error al cargar los datos base. Verificá tu sesión.', 'error');
    console.error('Error al cargar datos base:', error);
  } finally {
    cargando.value = false;
  }
});

// ─────────────────────────────────────────────
// PANEL 1: ROLES Y PERMISOS
// ─────────────────────────────────────────────
const cargarPermisosDeRol = async () => {
  if (!rolSeleccionado.value) return;
  cargandoPanel1.value = true;
  permisosDelRol.value = [];
  try {
    const response = await axios.get(`${baseURL}/roles/${rolSeleccionado.value}/permissions`);
    permisosDelRol.value = response.data.map(p => p.id);
  } catch (error) {
    mostrarMensaje('Error al cargar los permisos del rol.', 'error');
    console.error('Error al cargar permisos del rol:', error);
  } finally {
    cargandoPanel1.value = false;
  }
};

const guardarPermisosRol = async () => {
  guardandoPanel1.value = true;
  try {
    await axios.post(`${baseURL}/roles/${rolSeleccionado.value}/sync-permisos`, {
      permissions: permisosDelRol.value,
    });
    mostrarMensaje('✅ Permisos del rol guardados correctamente.');
  } catch (error) {
    mostrarMensaje('Error al guardar los permisos del rol.', 'error');
    console.error('Error al guardar permisos del rol:', error);
  } finally {
    guardandoPanel1.value = false;
  }
};

// ─────────────────────────────────────────────
// PANEL 2: USUARIOS Y ROLES
// ─────────────────────────────────────────────
const cargarRolesDeUsuario = async () => {
  if (!usuarioParaRol.value) return;
  cargandoPanel2.value = true;
  rolesDelUsuario.value = [];
  try {
    const response = await axios.get(`${baseURL}/users/${usuarioParaRol.value}/roles`);
    rolesDelUsuario.value = response.data.map(r => r.id);
  } catch (error) {
    mostrarMensaje('Error al cargar los roles del usuario.', 'error');
    console.error('Error al cargar roles del usuario:', error);
  } finally {
    cargandoPanel2.value = false;
  }
};

const guardarRolesUsuario = async () => {
  guardandoPanel2.value = true;
  try {
    await axios.post(`${baseURL}/users/${usuarioParaRol.value}/sync-roles`, {
      roles: rolesDelUsuario.value,
    });
    mostrarMensaje('✅ Roles del usuario guardados correctamente.');
  } catch (error) {
    mostrarMensaje('Error al guardar los roles del usuario.', 'error');
    console.error('Error al guardar roles del usuario:', error);
  } finally {
    guardandoPanel2.value = false;
  }
};

// ─────────────────────────────────────────────
// PANEL 3: USUARIOS Y PERMISOS DIRECTOS
// ─────────────────────────────────────────────
const cargarPermisosDeUsuario = async () => {
  if (!usuarioParaPermiso.value) return;
  cargandoPanel3.value = true;
  permisosDelUsuario.value = [];
  try {
    const response = await axios.get(`${baseURL}/users/${usuarioParaPermiso.value}/permissions`);
    permisosDelUsuario.value = response.data.map(p => p.id);
  } catch (error) {
    mostrarMensaje('Error al cargar los permisos del usuario.', 'error');
    console.error('Error al cargar permisos del usuario:', error);
  } finally {
    cargandoPanel3.value = false;
  }
};

const guardarPermisosUsuario = async () => {
  guardandoPanel3.value = true;
  try {
    await axios.post(`${baseURL}/users/${usuarioParaPermiso.value}/sync-permisos`, {
      permisos: permisosDelUsuario.value,
    });
    mostrarMensaje('✅ Permisos directos del usuario guardados correctamente.');
  } catch (error) {
    mostrarMensaje('Error al guardar los permisos del usuario.', 'error');
    console.error('Error al guardar permisos del usuario:', error);
  } finally {
    guardandoPanel3.value = false;
  }
};
</script>

<style scoped>
.manager-container {
  background-color: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1), 0 2px 4px -1px rgba(0,0,0,0.06);
  padding: 28px;
  margin: 10px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* HEADER */
.header {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 24px;
  padding-bottom: 20px;
  border-bottom: 2px solid #f3f4f6;
}
.header-icon { font-size: 2rem; }
.title { margin: 0; font-size: 1.5rem; color: #111827; font-weight: 700; }
.subtitle { margin: 4px 0 0; font-size: 0.85rem; color: #6b7280; }

/* LOADING GLOBAL */
.loading {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 24px;
  color: #6b7280;
  font-size: 0.95rem;
}
.spinner {
  width: 20px;
  height: 20px;
  border: 2px solid #e5e7eb;
  border-top-color: #4f46e5;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
  flex-shrink: 0;
}
@keyframes spin { to { transform: rotate(360deg); } }

/* ALERTAS */
.alerta {
  padding: 12px 16px;
  border-radius: 8px;
  margin-bottom: 20px;
  font-size: 0.9rem;
  font-weight: 500;
  animation: fadeIn 0.3s ease;
}
.alerta-success { background: #d1fae5; color: #065f46; border: 1px solid #a7f3d0; }
.alerta-error   { background: #fee2e2; color: #991b1b; border: 1px solid #fca5a5; }
@keyframes fadeIn { from { opacity: 0; transform: translateY(-6px); } to { opacity: 1; transform: translateY(0); } }

/* GRID */
.security-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 24px;
}

/* CARDS */
.card {
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 10px;
  padding: 20px;
  display: flex;
  flex-direction: column;
}
.card-header {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  margin-bottom: 20px;
}
.card-icon { font-size: 1.4rem; margin-top: 2px; }
.card-title { margin: 0; color: #374151; font-size: 1.05rem; font-weight: 600; }
.card-desc  { margin: 4px 0 0; color: #6b7280; font-size: 0.82rem; }

/* PANEL LOADING */
.panel-loading {
  font-size: 0.85rem;
  color: #6b7280;
  padding: 12px 0;
  text-align: center;
  font-style: italic;
}

/* FORM */
.form-group { margin-bottom: 16px; }
.form-label {
  display: block;
  font-weight: 600;
  margin-bottom: 6px;
  color: #374151;
  font-size: 0.85rem;
  text-transform: uppercase;
  letter-spacing: 0.03em;
}
.form-control {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 0.95rem;
  box-sizing: border-box;
  background: #ffffff;
  color: #111827;
  transition: border-color 0.2s;
  outline: none;
}
.form-control:focus { border-color: #4f46e5; box-shadow: 0 0 0 3px rgba(79,70,229,0.1); }

/* CHECKBOXES */
.checkbox-grid {
  display: flex;
  flex-direction: column;
  gap: 8px;
  max-height: 200px;
  overflow-y: auto;
  background: #ffffff;
  padding: 12px;
  border: 1px solid #e5e7eb;
  border-radius: 6px;
  flex: 1;
}
.checkbox-grid::-webkit-scrollbar { width: 4px; }
.checkbox-grid::-webkit-scrollbar-track { background: #f1f1f1; }
.checkbox-grid::-webkit-scrollbar-thumb { background: #d1d5db; border-radius: 2px; }

.checkbox-label {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 0.9rem;
  color: #4b5563;
  cursor: pointer;
  padding: 4px 6px;
  border-radius: 4px;
  transition: background 0.15s;
}
.checkbox-label:hover { background: #f3f4f6; }
.checkbox-input {
  width: 16px;
  height: 16px;
  accent-color: #4f46e5;
  cursor: pointer;
  flex-shrink: 0;
}
.checkbox-text { line-height: 1.3; }

/* BOTONES */
.btn {
  width: 100%;
  padding: 11px;
  border-radius: 7px;
  font-weight: 600;
  font-size: 0.9rem;
  cursor: pointer;
  border: none;
  transition: background-color 0.2s, opacity 0.2s, transform 0.1s;
}
.btn:active { transform: scale(0.98); }
.btn:disabled { opacity: 0.6; cursor: not-allowed; }
.btn-primary { background-color: #4f46e5; color: #ffffff; }
.btn-primary:hover:not(:disabled) { background-color: #4338ca; }
.mt-3 { margin-top: 16px; }
</style>