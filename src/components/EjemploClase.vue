<template>
  <div class="container">
    <h2>{{ editMode ? 'Editar Usuario' : 'Nuevo Usuario' }}</h2>

    <form @submit.prevent="saveUser" class="form-box">
      <input 
        v-model="currentUser.name" 
        placeholder="Nombre" 
        :class="{'error-border': errors.name}"
      />
      <p v-if="errors.name" class="error-text">El nombre es obligatorio</p>

      <input 
        v-model="currentUser.email" 
        type="email" 
        placeholder="Email" 
        :class="{'error-border': errors.email}"
      />
      <p v-if="errors.email" class="error-text">Email inválido o vacío</p>

      <button type="submit" class="btn-primary">
        {{ editMode ? 'Actualizar' : 'Crear Usuario' }}
      </button>
      <button v-if="editMode" @click="cancelEdit" type="button" class="btn-secondary">
        Cancelar
      </button>
    </form>
    <hr />

    <h2>Lista de Usuarios</h2>
    <table>
      <thead>
        <tr>
          <th>Nombre</th>
          <th>Email</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="user in users" :key="user.id">
          <td>{{ user.name }}</td>
          <td>{{ user.email }}</td>
          <td>
            <button @click="editUser(user)" class="btn-edit">Editar</button>
            <button @click="deleteUser(user.id)" class="btn-delete">Eliminar</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // Estado de la lista [cite: 82]
      users: [
        { id: 1, name: 'María', email: 'maria@email.com' },
        { id: 2, name: 'Juan', email: 'juan@email.com' },
        { id: 3, name: 'jose', email: 'maria@email.com' },
        { id: 4, name: 'luis', email: 'juan@email.com' },
        { id: 5, name: 'alberto', email: 'maria@email.com' },
        { id: 6, name: 'claudia', email: 'juan@email.com' }

      ],
      // Estado para el formulario [cite: 169, 311]
      currentUser: { id: null, name: '', email: '' },
      editMode: false,
      errors: { name: false, email: false },
      user: {
        name: '',
        email: '',
        role: 'admin',
        active: true
      }
    }
  },
  methods: {
    // Validar y Guardar (Alta o Edición) [cite: 215, 313, 314]
    saveUser() {
      // Validación simple [cite: 251, 252]
      this.errors.name = !this.currentUser.name;
      this.errors.email = !this.currentUser.email;

      if (this.errors.name || this.errors.email) return;

      if (this.editMode) {
        // Lógica de Edición [cite: 337, 339]
        const index = this.users.findIndex(u => u.id === this.currentUser.id);
        if (index !== -1) {
          this.users.splice(index, 1, { ...this.currentUser });
        }
      } else {
        // Lógica de Alta [cite: 196, 201]
        const newUser = {
          ...this.currentUser,
          id: Date.now() // Generar ID único [cite: 189]
        };
        this.users.push(newUser);
      }
      this.resetForm();
    },
    // Preparar edición [cite: 291, 294]
    editUser(user) {
      this.currentUser = { ...user }; // Copia para evitar reactividad inmediata
      this.editMode = true;
    },
    // Eliminar con confirmación [cite: 298, 299]
    deleteUser(id) {
      if (confirm('¿Estás seguro de eliminar este usuario?')) {
        this.users = this.users.filter(u => u.id !== id);
      }
    },
    cancelEdit() {
      this.resetForm();
    },
    resetForm() {
      this.currentUser = { id: null, name: '', email: '' };
      this.editMode = false;
      this.errors = { name: false, email: false };
    }
  }
}
</script>


<style scoped>
.container { max-width: 600px; margin: auto; font-family: sans-serif; }
.form-box { display: flex; flex-direction: column; gap: 10px; margin-bottom: 20px; }
input { padding: 8px; border: 1px solid #ccc; }
.error-border { border-color: red; }
.error-text { color: red; font-size: 0.8rem; margin: 0; }
table { width: 100%; border-collapse: collapse; }
th, td { border: 1px solid #ddd; padding: 10px; text-align: left; }
.btn-primary { background: #42b983; color: white; border: none; padding: 10px; cursor: pointer; }
.btn-edit { background: #ffc107; margin-right: 5px; }
.btn-delete { background: #f44336; color: white; }
button { cursor: pointer; border: none; padding: 5px 10px; border-radius: 4px; }
</style>