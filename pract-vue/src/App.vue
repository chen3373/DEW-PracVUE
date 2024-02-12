<template>
  <div id="app">
    <div v-if="!showCart">
      <div>
        <UserInput />
        <h1>Listado de productos</h1>
        <button @click="showCart = true, emit('cart-updated', updatedCart)" class="go-to-cart-btn">Ir al carrito ({{
          cart.length }})</button>
      </div>
      <ProductList :cart="cart" @product-added="addToCart" />
    </div>
    <Cart v-if="showCart" :cart="cart" @cart-updated="handleCartUpdate" @back-to-products="showCart = false" />
  </div>
</template>

<script setup>
import { ref } from 'vue';
import UserInput from './components/UserInput.vue';
import ProductList from './components/ProductList.vue';
import Cart from './components/Cart.vue';

const cart = ref([]);
const showCart = ref(false);

// No necesitas pasar cart como argumento a addToCart
const addToCart = (product) => {
  cart.value.push(product);
};

const handleCartUpdate = (updatedCart) => {
  cart = updatedCart;
}
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
</style>
