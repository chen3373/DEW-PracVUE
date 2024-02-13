<template>
    <div>
      <!-- Input para el nombre de usuario -->
      <input type="text" v-model="username" placeholder="Insertar nombre de usuario">
      <!-- Mostrar mensaje de bienvenida solo si el nombre de usuario no está vacío -->
      <div v-if="username !== ''" class="welcome-message">Bienvenido {{ username }}</div>
    </div>
  </template>
  
  <script setup>
  import { ref } from 'vue';
  
  // Definir una referencia reactiva para el nombre de usuario
  const username = ref('');
  </script>
  
  <style scoped>
  /* Estilos para el campo de entrada */
  input {
    width: 300px;
    padding: 10px;
    margin-bottom: 20px;
    font-size: 16px;
    border: 1px solid #ccc;
    border-radius: 5px;
  }
  
  /* Estilos para el mensaje de bienvenida */
  .welcome-message {
    font-size: 18px;
    font-weight: bold;
    color: #007bff;
  }
  </style>  