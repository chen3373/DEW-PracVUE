<template>
  <!-- Template del componente -->
  <div>
    <button @click="realizarPedido" class="realizar-pedido-button">Realizar Pedido</button>

    <!-- Campo de búsqueda -->
    <input type="text" v-model="searchText" placeholder="Buscar por nombre" class="search-input">
    
    <!-- Selector de categoría -->
    <div class="select-container">
      <label class="select-label">Categoría:</label>
      <select v-model="categorySelected" class="select-input">
        <option value="" class="select-option">Todas</option>
        <!-- Opciones de categoría dinámicas -->
        <option v-for="category in categories" :key="category" :value="category" class="select-option">{{ category }}</option>
      </select>
    </div>
    
    <!-- Tabla de productos -->
    <table class="product-table">
      <thead>
        <tr>
          <th>ID</th>
          <th>
            <!-- Encabezado de Nombre con opción de ordenamiento -->
            <span @click="sortByName = !sortByName" class="sortable-header">Nombre</span>
            <span v-if="sortByName.value" class="sort-indicator">&#8593;</span>
            <span v-else class="sort-indicator">&#8595;</span>
          </th>
          <th>
            <!-- Encabezado de Precio con opción de ordenamiento -->
            <span @click="sortByPrice = !sortByPrice" class="sortable-header">Precio</span>
            <span v-if="sortByPrice.value" class="sort-indicator">&#8593;</span>
            <span v-else class="sort-indicator">&#8595;</span>
          </th>
          <th>Categoría</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <!-- Renderizado de filas de productos -->
        <tr v-for="product in filteredProducts" :key="product.id">
          <td>{{ product.id }}</td>
          <td>{{ product.name }}</td>
          <td>{{ product.price }}</td>
          <td>{{ product.category }}</td>
          <td>
            <!-- Botones para añadir al carrito y eliminar productos -->
            <button @click="addToCart(product)">Añadir al carrito</button>
            <button @click="removeProduct(product)">Eliminar</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';

// URL base de la API
const API = "http://localhost:3000/api/";
const API_PEDIDOS = "http://localhost:3000/api/orders";

// Variables reactivas
const products = ref([]); // Lista de productos
const searchText = ref(''); // Texto de búsqueda
const categories = ref([]); // Lista de categorías
const categorySelected = ref(''); // Categoría seleccionada
const sortByName = ref(false); // Ordenar por nombre
const sortByPrice = ref(false); // Ordenar por precio

// Funciones de inicialización
const fetchProducts = async () => {
  try {
    const data = await getProducts();
    products.value = data;
  } catch (error) {
    console.error('Error al obtener los productos:', error);
  }
};

const fetchCategories = async () => {
  try {
    const data = await getCategories();
    categories.value = data;
  } catch (error) {
    console.error('Error al obtener las categorías:', error);
  }
};

// Ejecutar funciones de inicialización al montar el componente
onMounted(fetchProducts);
onMounted(fetchCategories);

// Funciones para obtener datos de la API
const getProducts = async () => {
  const response = await fetch(API + 'products');
  return response.json();
};

const getCategories = async () => {
  const response = await fetch(API + 'categories');
  return response.json();
};

// Función para añadir producto al carrito
const addToCart = (product) => {
  // Buscar si el producto ya está en el carrito
  const existingProduct = props.cart.find(item => item.id === product.id);

  // Si el producto ya está en el carrito, incrementar la cantidad
  if (existingProduct) {
    existingProduct.quantity += 1;
  } else {
    // Si el producto no está en el carrito, añadirlo con cantidad 1
    props.cart.push({ ...product, quantity: 1 });
  }

  // Find or create the updated cart entry
  const existingIndex = props.cart.findIndex(item => item.id === product.id);
  const updatedCart = existingIndex !== -1
    ? [...props.cart] // Copiar carrito para evitar mutación
    : [...props.cart, { ...product, quantity: 1 }];

  // Emit the updated cart data
  emit('update-cart', updatedCart);
};

// Función para eliminar producto
const removeProduct = async (product) => {
  try {
    const index = products.value.findIndex(item => item.id === product.id);
    if (index !== -1) {
      // Envía la petición de eliminar al servidor
      await deleteProduct(product.id);
      
      // Elimina el producto de la lista de productos
      products.value.splice(index, 1);

      // Guarda los productos eliminados en el almacenamiento local
      saveDeletedProductsToLocalStorage(products.value);

      // Check if the product is in the cart and remove it
      const cartIndex = props.cart.findIndex(item => item.id === product.id);
      if (cartIndex !== -1) {
        props.cart.splice(cartIndex, 1);
        emit('update-cart', props.cart); // Update the cart component
      }

      alert('Producto eliminado correctamente.');
    }
  } catch (error) {
    console.error('Error al eliminar el producto:', error);
  }
};

// Función para realizar pedido
const realizarPedido = async () => {
  // Verifica que haya un nombre de usuario
  if (!usuario.value) {
    alert("Por favor, introduce un nombre de usuario.");
    return;
  }

  // Construye el objeto de pedido con los datos del carrito
  const pedido = {
    usuario: usuario.value,
    productos: props.cart.map(item => ({ id: item.id, cantidad: item.quantity }))
  };

  // Envía el pedido al servidor
  const response = await fetch(API_PEDIDOS, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(pedido)
  });

  // Verifica si el pedido se realizó correctamente
  if (response.ok) {
    alert("Pedido realizado con éxito.");
    // Borra los datos del carrito
    props.cart = [];
    emit('update-cart', props.cart); // Update the cart component
  } else {
    throw new Error("Error al realizar el pedido.");
  }
};

// Computación de productos filtrados
const filteredProducts = computed(() => {
  return products.value.filter(product => {
    const matchesSearch = product.name.toLowerCase().includes(searchText.value.toLowerCase());
    const matchesCategory = categorySelected.value === '' || product.category === categorySelected.value;
    return matchesSearch && matchesCategory;
  }).sort((a, b) => {
    if (sortByName.value) {
      return (a.name < b.name) ? -1 : ((a.name > b.name) ? 1 : 0);
    } else if (sortByPrice.value) {
      return a.price - b.price;
    } else {
      return 0;
    }
  });
});

// Función para guardar productos eliminados en el almacenamiento local
const saveDeletedProductsToLocalStorage = (deletedProducts) => {
  localStorage.setItem('deletedProducts', JSON.stringify(deletedProducts));
};

// Función para eliminar producto de la API
const deleteProduct = async (productId) => {
  const response = await fetch(`${API}products/${productId}`, {
    method: 'DELETE',
  });
  if (!response.ok) {
    throw new Error('Error al eliminar producto del servidor');
  }
};
</script>

<style scoped>
/* Estilos CSS */
.realizar-pedido-button {
  margin-bottom: 20px;
}
.search-input {
  width: 100%;
  padding: 8px;
  margin-bottom: 16px;
  margin-top: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 5px;
}
.select-container {
  display: flex;
  align-items: center;
  margin-bottom: 16px;
}
.select-label {
  margin-right: 8px;
  font-weight: 500;
}
.select-input {
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 16px;
  width: 150px;
  appearance: none;
  background-color: #fff;
}
.select-option {
  padding: 8px;
  font-size: 16px;
  color: #333;
  cursor: pointer;
}
.select-option:hover {
  background-color: #f2f2f2;
}
.sortable-header {
  cursor: pointer;
}
.sort-indicator {
  font-size: 12px;
  margin-left: 5px;
}
.product-table {
  width: 100%;
  border-collapse: collapse;
}
.product-table th,
.product-table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}
.product-table th {
  background-color: #f2f2f2;
}
</style>
