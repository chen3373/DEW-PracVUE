<template>
    <div>
        <input type="text" v-model="username" placeholder="Insertar nombre de usuario">
        <div v-if="username !== ''">Bienvenido {{ username }}</div>
    </div>
</template>
  
<script setup>
import { ref } from 'vue';
const username = ref('');

</script>
  
<style scoped>
.input-field {
    width: 300px;
    padding: 10px;
    margin-bottom: 20px;
    font-size: 16px;
    border: 1px solid #ccc;
    border-radius: 5px;
}

.welcome-message {
    font-size: 18px;
    font-weight: bold;
    color: #007bff;
}
</style>
  