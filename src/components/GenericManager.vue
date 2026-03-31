<template>
  <div class="manager-container">
    <div class="header">
      <h1 class="title">Gestión de <span class="capitalize">{{ recursoActive }}</span></h1>
      <button v-if="configRecurso.canAdd" class="btn btn-primary" @click="abrirModalCrear">
        + Nuevo {{ recursoActive }} 
      </button>
    </div>

    <div v-if="cargando" class="loading">
      <div class="spinner"></div> Cargando datos...
    </div>

    <div class="table-responsive" v-else>
      <table class="styled-table">
        <thead>
          <tr>
            <th v-for="col in columnasLista" :key="col.key">{{ col.label }}</th>
            <th class="actions-col" v-if="configRecurso.canEdit || configRecurso.canDelete">Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="datos.length === 0">
            <td :colspan="columnasLista.length + (configRecurso.canEdit || configRecurso.canDelete ? 1 : 0)" class="text-center">No hay registros disponibles.</td>
          </tr>
          <tr v-for="item in datos" :key="item.id">
            <td v-for="col in columnasLista" :key="col.key">
              {{ obtenerValor(item, col) }}
            </td>
            <td class="actions-cell" v-if="configRecurso.canEdit || configRecurso.canDelete">
              <button v-if="configRecurso.canEdit" class="btn btn-sm btn-edit" @click="editar(item)">✎ Editar</button>
              <button v-if="configRecurso.canDelete" class="btn btn-sm btn-delete" @click="abrirModalEliminar(item.id)">🗑 Eliminar</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Modal Crear / Editar -->
    <div v-if="mostrarModalForm" class="modal-overlay" @click.self="cerrarModalForm">
      <div class="modal-content">
        <h2 class="modal-title">{{ modoEdicion ? 'Editar' : 'Nuevo' }} {{ recursoActive }}</h2>
        
        <form @submit.prevent="guardar">
          <div class="form-group" v-for="col in columnasFormulario" :key="col.key">
            <template v-if="col.type === 'checkbox'">
              <label class="form-check-label" style="display: flex; align-items: center; gap: 8px;">
                <input 
                  type="checkbox"
                  v-model="formulario[col.formKey || col.key]"
                  class="form-check-input"
                  :disabled="col.readonly"
                />
                {{ col.label }}
              </label>
            </template>
            <template v-else>
              <label class="form-label">{{ col.label }}</label>
              
              <select v-if="col.type === 'select' && col.optionsResource"
                v-model="formulario[col.formKey || col.key]"
                class="form-control"
                required
              >
                <option value="" disabled>Seleccione {{ col.label.toLowerCase() }}</option>
                <option v-for="opt in opciones[col.optionsResource] || []" :key="opt[col.optionValue]" :value="opt[col.optionValue]">
                  {{ opt[col.optionLabel] }}
                </option>
              </select>
              
              <input v-else
                v-model="formulario[col.formKey || col.key]" 
                class="form-control" 
                :type="col.type || 'text'"
                :placeholder="'Ingrese ' + col.label.toLowerCase()"
                :readonly="col.readonly"
                required 
              />
            </template>
          </div>
          
          <div class="modal-actions">
            <button type="button" class="btn btn-secondary" @click="cerrarModalForm">Cancelar</button>
            <button type="submit" class="btn btn-primary">
              {{ modoEdicion ? 'Actualizar' : 'Guardar' }}
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Modal Confirmar Eliminación -->
    <div v-if="mostrarModalEliminar" class="modal-overlay" @click.self="cerrarModalEliminar">
      <div class="modal-content modal-sm">
        <h2 class="modal-title text-danger">⚠️ Confirmar Eliminación</h2>
        <p class="modal-text">¿Estás seguro de que deseas eliminar este registro? Esta acción no se puede deshacer.</p>
        <div class="modal-actions">
          <button type="button" class="btn btn-secondary" @click="cerrarModalEliminar">Cancelar</button>
          <button type="button" class="btn btn-delete" @click="confirmarEliminar">Sí, Eliminar</button>
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, watch, defineProps, computed } from 'vue';
import axios from 'axios';

const props = defineProps({ recursoActive: { type: String, required: true } });
const datos = ref([]);
const columnas = ref([]);
const cargando = ref(false);

// Estados para Modales
const mostrarModalForm = ref(false);
const mostrarModalEliminar = ref(false);
const modoEdicion = ref(false);
const formulario = ref({});
const itemAEliminar = ref(null);
const opciones = ref({});

// Configuración de permisos por recurso
const configRecurso = computed(() => {
  const configuraciones = {
    clientes: { canAdd: true, canEdit: true, canDelete: true },
    productos: { canAdd: true, canEdit: true, canDelete: true },
    ordenes: { canAdd: true, canEdit: true, canDelete: true },
    'detalle-ordenes': { canAdd: false, canEdit: false, canDelete: false },
    envios: { canAdd: false, canEdit: false, canDelete: false }
  };
  return configuraciones[props.recursoActive] || { canAdd: true, canEdit: true, canDelete: true };
});

// Filtramos el 'id' para no mostrarlo en el formulario de creación/edición
const columnasFormulario = computed(() => {
  return columnas.value.filter(col => col.key !== 'id' && col.showInForm !== false);
});

const columnasLista = computed(() => {
  return columnas.value.filter(col => col.showInList !== false);
});

// Configuración de columnas por entidad [cite: 148, 149]
const actualizarColumnas = (recurso) => {
  const configs = {
    clientes: [
      { key: 'id', label: 'ID' },
      { key: 'name', label: 'Nombre' },
      { key: 'email', label: 'Email', type: 'email' }
    ],
    productos: [
      { key: 'id', label: 'ID' },
      { key: 'nombre', label: 'Producto' },
      { key: 'precio', label: 'Precio', type: 'number' },
      { key: 'stock', label: 'Stock', type: 'number' }
    ],
    ordenes: [
      // Para listado
      { key: 'id', label: 'ID', showInForm: false },
      { key: 'cliente.name', label: 'Cliente', showInForm: false },
      { key: 'total', label: 'Total', type: 'number', showInForm: false },
      { key: 'envio', label: 'Envios', type: 'checkbox',  showInForm: false },
      // Para formulario
      { key: 'clientes_id', label: 'Cliente', type: 'select', optionsResource: 'clientes', optionValue: 'id', optionLabel: 'name', showInList: false },
      { key: 'producto_id', label: 'Producto', type: 'select', optionsResource: 'productos', optionValue: 'id', optionLabel: 'nombre', showInList: false },
      { key: 'cantidad', label: 'Cantidad', type: 'number', default: 1, showInList: false },
      { key: 'envio', label: 'Envío', type: 'checkbox', default: false, showInList: false }
    ],
    'detalle-ordenes': [
      { key: 'id', label: 'ID' },
      { key: 'ordene_id', label: 'Orden N°' },
      { key: 'producto.nombre', label: 'Producto' },
      { key: 'cantidad', label: 'Cantidad', type: 'number' },
      { key: 'precio_unitario', label: 'Precio Calculado', type: 'number' }
    ],
    envios: [
      { key: 'id', label: 'ID' },
      { key: 'cliente.name', label: 'Cliente', showInForm: false },
      { key: 'ordene_id', label: 'Orden N°' },
      { key: 'fecha_envio', label: 'Fecha de Envío', type: 'date' }
    ]
  };
  columnas.value = configs[recurso] || [];
};

// Función dinámica para consumir la API de Laravel [cite: 136, 140]
const baseURL = 'http://localhost:8081/api';

// Función para resolver rutas anidadas (ej: "cliente.name")
const obtenerValor = (objeto, col) => {
  const path = typeof col === 'string' ? col : col.key;
  let valor = path.split('.').reduce((acc, prop) => (acc && acc[prop] !== undefined) ? acc[prop] : '', objeto);
  
  // Si la columna es de tipo fecha, formateamos a dd/mm/aaaa
  if (col && col.type === 'date' && valor) {
    const match = String(valor).match(/^(\d{4})-(\d{2})-(\d{2})/);
    if (match) {
      return `${match[3]}/${match[2]}/${match[1]}`;
    }
  }

  return valor;
};

const fetchData = async () => {
  cargando.value = true;
  try {
    const response = await axios.get(`${baseURL}/${props.recursoActive}`);
    datos.value = response.data;
  } catch (error) {
    console.error("Error al cargar:", error);
  } finally {
    cargando.value = false;
  }
};

const cargarOpciones = async (recurso) => {
  if (!opciones.value[recurso]) {
    try {
      const response = await axios.get(`${baseURL}/${recurso}`);
      opciones.value = { ...opciones.value, [recurso]: response.data };
    } catch (error) {
      console.error(`Error al cargar opciones para ${recurso}:`, error);
    }
  }
};

const abrirModalCrear = () => {
  formulario.value = {};
  columnasFormulario.value.forEach(col => {
    let def = col.default !== undefined ? col.default : '';
    if (col.type === 'checkbox' && col.default === undefined) def = false;
    formulario.value[col.formKey || col.key] = def; // Aplica el valor por defecto si existe, o lo deja vacío
  });
  modoEdicion.value = false;
  mostrarModalForm.value = true;
};

const editar = async (item) => {
  modoEdicion.value = true;
  mostrarModalForm.value = true;

  let formData = { ...item };

  if (props.recursoActive === 'ordenes') {
    // Mapear el ID del cliente desde el objeto anidado
    if (item.cliente) {
      formData.cliente_id = item.cliente.id;
    }

    // Para editar, necesitamos producto y cantidad, que no están en la lista principal.
    // Los buscamos en los detalles de la orden. Usamos 'ordene_id' por consistencia.
    try {
      const detallesResponse = await axios.get(`${baseURL}/detalle-ordenes`, { params: { ordene_id: item.id } });
      if (detallesResponse.data && detallesResponse.data.length > 0) {
        const primerDetalle = detallesResponse.data[0];
        formData.producto_id = primerDetalle.producto_id;
        formData.cantidad = primerDetalle.cantidad;
      }
    } catch (error) {
      console.error(`Error al buscar detalles para la orden ${item.id}:`, error);
    }
  }
  
  formulario.value = formData;
};

const cerrarModalForm = () => {
  mostrarModalForm.value = false;
};

const guardar = async () => {
  try {
    if (modoEdicion.value) {
      await axios.put(`${baseURL}/${props.recursoActive}/${formulario.value.id}`, formulario.value);
    } else {
      await axios.post(`${baseURL}/${props.recursoActive}`, formulario.value);
    }
    fetchData(); // Refrescar datos
    cerrarModalForm(); // Cerrar el modal
  } catch (error) {
    console.error("Error al guardar:", error);
    alert("Hubo un error al guardar el registro.");
  }
};

const abrirModalEliminar = (id) => {
  itemAEliminar.value = id;
  mostrarModalEliminar.value = true;
};

const cerrarModalEliminar = () => {
  mostrarModalEliminar.value = false;
  itemAEliminar.value = null;
};

const confirmarEliminar = async () => {
  if (itemAEliminar.value) {
    try {
      await axios.delete(`${baseURL}/${props.recursoActive}/${itemAEliminar.value}`);
      fetchData();
      cerrarModalEliminar();
    } catch (error) {
      console.error("Error al eliminar:", error);
      alert("Hubo un error al intentar eliminar.");
    }
  }
};

// El Watch detecta el cambio de recurso y dispara la actualización [cite: 130, 134]
watch(() => props.recursoActive, (nuevo) => {
  actualizarColumnas(nuevo);
  fetchData();
  
  columnas.value.forEach(col => {
    if (col.type === 'select' && col.optionsResource) {
      cargarOpciones(col.optionsResource);
    }
  });
}, { immediate: true });
</script>

<style scoped>
.manager-container {
  background-color: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  padding: 24px;
  margin: 10px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
  padding-bottom: 16px;
  border-bottom: 2px solid #f3f4f6;
}

.title {
  margin: 0;
  font-size: 1.5rem;
  color: #111827;
  font-weight: 600;
}

.capitalize {
  text-transform: capitalize;
}

/* Estilos de Tabla */
.table-responsive {
  overflow-x: auto;
}

.styled-table {
  width: 100%;
  border-collapse: collapse;
  text-align: left;
}

.styled-table th {
  background-color: #f9fafb;
  color: #4b5563;
  font-weight: 600;
  padding: 12px 16px;
  border-bottom: 2px solid #e5e7eb;
  text-transform: uppercase;
  font-size: 0.85rem;
  letter-spacing: 0.05em;
}

.styled-table td {
  padding: 14px 16px;
  border-bottom: 1px solid #f3f4f6;
  color: #374151;
}

.styled-table tr:hover {
  background-color: #f9fafb;
}

.actions-col {
  width: 180px;
  text-align: center;
}

.actions-cell {
  display: flex;
  gap: 8px;
  justify-content: center;
}

.text-center { text-align: center; }
.text-danger { color: #dc2626; }

/* Botones */
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 8px 16px;
  border-radius: 6px;
  font-weight: 500;
  font-size: 0.9rem;
  cursor: pointer;
  border: none;
  transition: all 0.2s ease;
}

.btn-sm { padding: 6px 12px; font-size: 0.8rem; }

.btn-primary { background-color: #4f46e5; color: #ffffff; }
.btn-primary:hover { background-color: #4338ca; }

.btn-edit { background-color: #f59e0b; color: #ffffff; }
.btn-edit:hover { background-color: #d97706; }

.btn-delete { background-color: #ef4444; color: #ffffff; }
.btn-delete:hover { background-color: #dc2626; }

.btn-secondary { background-color: #f3f4f6; color: #374151; border: 1px solid #d1d5db; }
.btn-secondary:hover { background-color: #e5e7eb; }

/* Modales */
.modal-overlay {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background-color: rgba(17, 24, 39, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  backdrop-filter: blur(2px);
}

.modal-content {
  background-color: #ffffff;
  padding: 24px 32px;
  border-radius: 12px;
  width: 100%;
  max-width: 500px;
  box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
  animation: modal-fade-in 0.3s ease-out;
}

.modal-sm { max-width: 400px; text-align: center; }

.modal-title {
  margin-top: 0;
  margin-bottom: 20px;
  font-size: 1.4rem;
  color: #111827;
}

.modal-text {
  color: #4b5563;
  margin-bottom: 24px;
  line-height: 1.5;
}

.form-group { margin-bottom: 16px; text-align: left; }

.form-label {
  display: block;
  font-weight: 500;
  margin-bottom: 6px;
  color: #374151;
}

.form-control {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 0.95rem;
  box-sizing: border-box;
  transition: border-color 0.2s;
}

.form-control:focus {
  outline: none;
  border-color: #4f46e5;
  box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.1);
}

.form-control[readonly] {
  background-color: #f9fafb;
  color: #6b7280;
  cursor: not-allowed;
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 24px;
}

/* Utilidades */
.loading {
  display: flex; align-items: center; justify-content: center; gap: 10px;
  color: #6b7280; padding: 40px; font-weight: 500;
}

.spinner {
  border: 3px solid rgba(0,0,0,0.1); border-left-color: #4f46e5;
  border-radius: 50%; width: 20px; height: 20px; animation: spin 1s linear infinite;
}

@keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
@keyframes modal-fade-in {
  from { opacity: 0; transform: translateY(-20px) scale(0.95); }
  to { opacity: 1; transform: translateY(0) scale(1); }
}
</style>