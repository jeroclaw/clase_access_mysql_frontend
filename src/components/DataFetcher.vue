<template>
  <div class="data-list-container">
    <h2>Nuestros Usuarios</h2>
    <p v-if="loading">Cargando datos...</p>
    <p v-else-if="error" class="error-message">Error: {{ error }}</p>
    <p v-else-if="users.length === 0">No hay datos disponibles.</p>
    <ul v-else class="user-list">
      <li v-for="user in users" :key="user.id" class="user-item">
        <div class="user-id">ID: {{ user.id }}</div>
        <div class="user-name">Nombre: <strong>{{ user.name }}</strong></div>
        <div class="user-email">Email: <a :href="'mailto:' + user.email">{{ user.email }}</a></div>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      users: [],
      loading: true,
      error: null,
    };
  },
  created() {
    axios.defaults.baseURL = 'http://localhost:8000/api'; // Ajusta a tu URL de Laravel
    this.fetchUsers();
  },
  methods: {
    fetchUsers() {
      axios.get('/clientes') // Asumiendo que tienes una ruta /api/users en Laravel
        .then(response => {
          this.users = response.data;
          this.loading = false;
        })
        .catch(error => {
          this.error = error.message || 'Error al cargar usuarios.';
          this.loading = false;
          console.error('Hubo un error:', error);
        });
    },
  },
};
</script>

<style>
.data-list-container {
  font-family: Arial, sans-serif;
  max-width: 800px;
  margin: 20px auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.data-list-container h2 {
  color: #333;
  text-align: center;
  margin-bottom: 25px;
}

.user-list {
  list-style: none;
  padding: 0;
}

.user-item {
  background-color: #fff;
  border: 1px solid #ddd;
  border-radius: 5px;
  margin-bottom: 15px;
  padding: 15px 20px;
  display: flex;
  flex-direction: column;
  gap: 5px;
  transition: transform 0.2s ease-in-out;
}

.user-item:hover {
  transform: translateY(-3px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.08);
}

.user-id {
  font-size: 0.9em;
  color: #666;
}

.user-name {
  font-size: 1.1em;
  color: #0056b3;
}

.user-email a {
  color: #28a745;
  text-decoration: none;
}

.user-email a:hover {
  text-decoration: underline;
}

.error-message {
  color: #dc3545;
  text-align: center;
  font-weight: bold;
}
</style>
