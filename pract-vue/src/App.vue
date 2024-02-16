<template>
  <!-- Contenedor principal de la aplicación -->
  <div id="app">
    <!-- Contenido mostrado cuando no se muestra el carrito -->
    <div v-if="!showCart">
      <!-- Sección para ingresar nombre de usuario -->
      <div>
        <!-- Input para el nombre de usuario -->
        <input type="text" v-model="userName" class="inputUser" placeholder="Insertar nombre de usuario">
        <!-- Mostrar mensaje de bienvenida solo si el nombre de usuario no está vacío -->
        <div v-if="userName !== ''" class="welcome-message">Bienvenido {{ userName }}</div>
      </div>
      <h1>Listado de productos</h1>
      <div>
        <!-- Botón para ir al carrito -->
        <button @click="showCart = true" class="go-to-cart-btn">Ir al carrito ({{ cart.length }})</button>
        <!-- Botón para enviar el pedido -->
        <button @click="sendOrder" class="send-order-btn">Enviar Pedido</button>
      </div>
    </div>
    <!-- Componente ProductList que muestra la lista de productos -->
    <ProductList :cart="cart" />
    <!-- Componente Cart que muestra el carrito de compras -->
    <Cart v-if="showCart" :cart="cart" @back-to-products="showCart = false" />
    <!-- Botón para eliminar el carrito -->
    <button @click="removeShoppingCart" class="remove-cart-btn">Eliminar Carrito</button>
  </div>
</template>

<script setup>
// Importar funciones y objetos necesarios de Vue
import { ref, watch, onMounted } from 'vue';
import ProductList from './components/ProductList.vue';
import Cart from './components/Cart.vue';

// Definir datos reactivos
const userName = ref('Chen');
const cart = ref([]);
const showCart = ref(false);

// Observar cambios en el carrito y almacenarlo en local storage
watch(cart, (newCart) => {
  localStorage.setItem('cart', JSON.stringify(newCart));
}, { deep: true });

// Función para cargar el carrito desde local storage al iniciar la aplicación
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

// Función para enviar el pedido
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
/* Estilos específicos del componente */
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
