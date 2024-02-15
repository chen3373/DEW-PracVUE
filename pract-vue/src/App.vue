<template>
  <div id="app">
    <div v-if="!showCart">
      <div>
        <div>
          <!-- Input para el nombre de usuario -->
          <input type="text" v-model="userName" class="inputUser" placeholder="Insertar nombre de usuario">
          <!-- Mostrar mensaje de bienvenida solo si el nombre de usuario no está vacío -->
          <div v-if="userName !== ''" class="welcome-message">Bienvenido {{ userName }}</div>
        </div>
        <h1>Listado de productos</h1>
        <div>
          <button @click="showCart = true" class="go-to-cart-btn">Ir al carrito ({{ cart.length }})</button>
          <button @click="sendOrder" class="send-order-btn">Enviar Pedido</button>
        </div>
      </div>
      <ProductList :cart="cart" />
    </div>
    <Cart v-if="showCart" :cart="cart" @back-to-products="showCart = false" />
    <!-- Botón para eliminar el carrito -->
    <button @click="removeShoppingCart" class="remove-cart-btn">Eliminar Carrito</button>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue';
import ProductList from './components/ProductList.vue';
import Cart from './components/Cart.vue';

const userName = ref('Chen');
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

// Llama a loadLocalCart cuando el componente se monta
onMounted(() => {
  loadLocalCart();
});

const sendOrder = async () => {
  try {
    const userName = 'Juan'; // Nombre fijo para simplificar
    const cartIds = cart.value.map(item => ({ id: item.id, amount: item.quantity }));
    const requestOptions = {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        user: userName,
        products: cartIds
      })
    }
    const response = await fetch("http://localhost:3000/api/orders", requestOptions);
    if (response.ok && response.status === 201) {
      alert('Pedido realizado con éxito');
      removeShoppingCart();
    } else {
      throw ('Error al realizar la petición.')
    }
  } catch (error) {
    console.error('Error al realizar la petición: ', error);
    alert('Error al realizar el pedido');
  }
};


</script>

<style scoped>
#app {
  padding: 20px;
}

.main-content {
  margin-bottom: 20px;
}

.go-to-cart-btn,
.send-order-btn {
  margin-bottom: 5px;
  margin-right: 5px;
  padding: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.go-to-cart-btn:hover,
.send-order-btn:hover {
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

/* Estilos para el campo de entrada */
.inputUser {
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
