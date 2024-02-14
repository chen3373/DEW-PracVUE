<template>
    <div class="cart">
      <!-- Botón para volver a la lista de productos -->
      <button @click="goBackToProducts">Volver a la lista de productos</button>
      <h2>Carrito de Compras</h2>
      <!-- Mostrar un mensaje si el carrito está vacío -->
      <p v-if="!cart.length">No hay productos en el carrito.</p>
      <!-- Tabla para mostrar los elementos del carrito -->
      <table class="cart-table">
        <thead>
          <tr>
            <th style="width: 200px;">Nombre</th>
            <th style="width: 100px;">Precio</th>
            <th style="width: 150px;">Cantidad</th>
            <th style="width: 100px;">Total</th>
            <th style="width: 150px;">Acciones</th>
          </tr>
        </thead>
        <tbody>
          <!-- Iterar sobre los elementos del carrito -->
          <tr v-for="(item, index) in cart" :key="index">
            <td style="width: 200px;">{{ item.name }}</td>
            <td style="width: 100px;">{{ item.price }}</td>
            <td class="quantity" style="width: 150px;">
              <!-- Botones para incrementar y decrementar la cantidad -->
              <button @click="decrementQuantity(index)">-</button>
              <input type="number" v-model.number="item.quantity" min="0" @blur="checkQuantity(index)" @input="checkInput(index)" />
              <button @click="incrementQuantity(index)">+</button>
            </td>
            <td style="width: 100px;">{{ item.price * item.quantity }}</td>
            <td class="actions" style="width: 150px;">
              <!-- Botón para eliminar el producto del carrito -->
              <button @click="removeFromCart(item)">Eliminar</button>
            </td>
          </tr>
        </tbody>
      </table>
      <!-- Mostrar el total del pedido y botones para vaciar el carrito y eliminar el carrito almacenado -->
      <div class="cart-total">
        <p>Total del Pedido: {{ total }}</p>
      </div>
    </div>
  </template>
  
  <script setup>
  import { computed, defineProps, defineEmits, onMounted } from 'vue';
  
  const emit = defineEmits(['back-to-products', 'empty-cart']);
  
  // Definir las propiedades del componente
  const props = defineProps({
    cart: {
      type: Array,
      default: () => [],
    },
  });
  
  // Calcular el total del carrito
  const total = computed(() => {
    return props.cart.reduce((acc, item) => acc + (item.price * item.quantity), 0);
  });
  
  // Remover un producto del carrito
  const removeFromCart = (product) => {
    const index = props.cart.findIndex(item => item.id === product.id);
    if (index !== -1) {
      props.cart.splice(index, 1);
      saveCartToLocalStorage();
    }
    // Enviar solicitud al servidor para eliminar el producto
    deleteProductOnServer(product.id)
      .then(() => {
        console.log('Producto eliminado del servidor con éxito.');
      })
      .catch(error => {
        console.error('Error al eliminar producto del servidor:', error);
      });
  };
  

  
  
  // Función para volver a la lista de productos
  const goBackToProducts = () => {
    emit('back-to-products');
  };
  
  // Decrementar la cantidad de un producto en el carrito
  const decrementQuantity = (index) => {
    if (props.cart[index].quantity > 0) {
      props.cart[index].quantity--;
      if (props.cart[index].quantity === 0) {
        removeFromCart(props.cart[index]);
      }
      saveCartToLocalStorage();
    }
  };
  
  // Incrementar la cantidad de un producto en el carrito
  const incrementQuantity = (index) => {
    props.cart[index].quantity++;
    saveCartToLocalStorage();
  };
  
  // Verificar la cantidad de un producto en el carrito
  const checkQuantity = (index) => {
    if (props.cart.length > 0 && props.cart[index].quantity <= 0) {
      removeFromCart(props.cart[index]);
    }
  };
  
  // Verificar la entrada de cantidad de un producto en el carrito
  const checkInput = (index) => {
    if (props.cart.length > 0 && props.cart[index].quantity <= 0) {
      removeFromCart(props.cart[index]);
    }
  };
  
  // Guardar el carrito en el almacenamiento local
  const saveCartToLocalStorage = () => {
    try {
      const serializedCart = JSON.stringify(props.cart);
      localStorage.setItem('cart', serializedCart);
    } catch (error) {
      console.error('Error saving cart to local storage:', error);
    }
  };
  
  // Eliminar el carrito almacenado en el almacenamiento local
  const clearLocalStorage = () => {
    localStorage.removeItem('cart');
  };
  
  // Cargar el carrito desde el almacenamiento local al montar el componente
  onMounted(() => {
    try {
      const savedCart = localStorage.getItem('cart');
      if (savedCart) {
        const parsedCart = JSON.parse(savedCart);
        if (Array.isArray(parsedCart)) {
          props.cart = parsedCart;
        } else {
          console.error('Error: Invalid cart data found in localStorage');
        }
      } else {
        props.cart = []; // Ensure an empty array as default
      }
    } catch (error) {
      console.error('Error loading cart from local storage:', error);
    }
  });
  
  // Función para manejar la actualización del carrito
  const handleUpdateCart = (updatedCart) => {
    props.cart = updatedCart;
    saveCartToLocalStorage(); // Actualizar el almacenamiento local también
  };
  
  // Eliminar un producto del servidor
  const deleteProductOnServer = async (productId) => {
    try {
      const response = await fetch(`${API}products/${productId}`, {
        method: 'DELETE',
      });
      if (!response.ok) {
        throw new Error('Error deleting product from server.');
      }
      return response.json();
    } catch (error) {
      throw error;
    }
  };
  </script>
  
  <style scoped>
  .cart-table {
    width: 100%;
    border-collapse: collapse;
  }
  
  .cart-table th,
  .cart-table td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
  }
  
  .cart-table th {
    background-color: #f2f2f2;
  }
  
  .cart-total {
    border-top: 1px solid #ccc;
    padding: 10px;
    text-align: right;
  }
  
  .cart-table .actions {
    text-align: center;
  }
  
  .cart-table .quantity {
    text-align: center;
  }
  </style>