<template>
  <div id="app">
    <div v-if="!showCart">
      <div>
        <UserInput />
        <h1>Listado de productos</h1>
        <button @click="showCart = true" class="go-to-cart-btn">Ir al carrito ({{ cart.length }})</button>
      </div>
      <ProductList :cart="cart" @product-added="addToCart" />
    </div>
    <Cart v-if="showCart" :cart="cart" @back-to-products="showCart = false" />
    <!-- Botón para eliminar el carrito -->
    <button @click="removeShoppingCart" class="remove-cart-btn">Eliminar Carrito</button>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue';
import UserInput from './components/UserInput.vue';
import ProductList from './components/ProductList.vue';
import Cart from './components/Cart.vue';

const cart = ref([]);
const showCart = ref(false);

// Watcher para observar cambios en cart y almacenarlo en local storage
watch(cart, (newCart) => {
  localStorage.setItem('cart', JSON.stringify(newCart));
}, { deep: true });

// Función para cargar el carrito desde el local storage al iniciar la aplicación
const loadLocalCart = () => {
  const savedCart = localStorage.getItem('cart');
  if (savedCart) {
    cart.value = JSON.parse(savedCart);
  }
};

// Función para eliminar el carrito
const removeShoppingCart = () => {
  cart.value = [];
};
</script>

<style scoped>
#app {
  padding: 20px;
}

.main-content {
  margin-bottom: 20px;
}

.go-to-cart-btn {
  margin-bottom: 5px;
  padding: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.go-to-cart-btn:hover {
  background-color: #0056b3;
}

.remove-cart-btn {
  margin-top: 10px;
  padding: 10px;
  background-color: #dc3545;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.remove-cart-btn:hover {
  background-color: #c82333;
}
</style>
