<template>
  <!-- Mostrar un mensaje de éxito cuando se realiza una acción exitosa -->
  <div v-if="successMessage" class="success-message">
    {{ successMessage }}
  </div>
  <div>
    <!-- Input para buscar productos por nombre -->
    <input type="text" v-model="searchText" placeholder="Buscar por nombre" class="search-input">
    <!-- Select para filtrar por categoría -->
    <div class="select-container">
      <label class="select-label">Categoría:</label>
      <select v-model="categorySelected" class="select-input">
        <!-- Opción para mostrar todos los productos -->
        <option value="" class="select-option">Todas</option>
        <!-- Iterar sobre las categorías disponibles -->
        <option v-for="category in categories" :key="category" :value="category" class="select-option">{{ category }}</option>
      </select>
    </div>
    <!-- Tabla para mostrar los productos -->
    <table class="product-table">
      <thead>
        <tr>
          <th>ID</th>
          <!-- Encabezado para ordenar por nombre -->
          <th>
            <span @click="sortByName = !sortByName" class="sortable-header">Nombre</span>
            <!-- Indicador de orden ascendente/descendente para nombre -->
            <span v-if="sortByName.value" class="sort-indicator">&#8593;</span>
            <span v-else class="sort-indicator">&#8595;</span>
          </th>
          <!-- Encabezado para ordenar por precio -->
          <th>
            <span @click="sortByPrice = !sortByPrice" class="sortable-header">Precio</span>
            <!-- Indicador de orden ascendente/descendente para precio -->
            <span v-if="sortByPrice.value" class="sort-indicator">&#8593;</span>
            <span v-else class="sort-indicator">&#8595;</span>
          </th>
          <th>Categoría</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <!-- Iterar sobre los productos filtrados -->
        <tr v-for="product in filteredProducts" :key="product.id" :class="{ alcoholic: product.category == 'Alcoholic' }">
          <td>{{ product.id }}</td>
          <td>{{ product.name }}</td>
          <td>{{ product.price }}</td>
          <td>{{ product.category }}</td>
          <!-- Botones para agregar al carrito y eliminar producto -->
          <td>
            <button @click="addToCart(product)">Añadir al carrito</button>
            <button @click="removeProduct(product)">Eliminar</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
// Importar funciones y objetos necesarios de Vue
import { ref, onMounted, computed, defineProps, defineEmits } from 'vue';

// URL base para las llamadas a la API
const API = "http://localhost:3000/api/";

// Definir eventos emitidos por el componente
const emit = defineEmits(['product-added', 'update-cart']);

// Variables reactivas
const products = ref([]); // Lista de productos
const searchText = ref(''); // Texto de búsqueda
const categories = ref([]); // Lista de categorías
const categorySelected = ref(''); // Categoría seleccionada para filtrar
const sortByName = ref(false); // Flag para ordenar por nombre
const sortByPrice = ref(false); // Flag para ordenar por precio
const props = defineProps({ // Props recibidos por el componente
  cart: Array
});
const successMessage = ref(''); // Mensaje de éxito

// Método para obtener productos desde la API
const fetchProducts = async () => {
  try {
    const data = await getProducts();
    products.value = data;
  } catch (error) {
    console.error('Error al obtener los productos:', error);
  }
};

// Método para obtener categorías desde la API
const fetchCategories = async () => {
  try {
    const data = await getCategories();
    categories.value = data;
  } catch (error) {
    console.error('Error al obtener las categorías:', error);
  }
};

// Ejecutar métodos de obtención de productos y categorías al montar el componente
onMounted(fetchProducts);
onMounted(fetchCategories);

// Función para obtener productos desde la API
const getProducts = async () => {
  const response = await fetch(API + 'products');
  return response.json();
};

// Función para obtener categorías desde la API
const getCategories = async () => {
  const response = await fetch(API + 'categories');
  return response.json();
};

// Función para añadir un producto al carrito
const addToCart = (product) => {
  const existingProduct = props.cart.find(item => item.id === product.id);

  if (existingProduct) {
    existingProduct.quantity += 1;
  } else {
    props.cart.push({ ...product, quantity: 1 });
  }

  const existingIndex = props.cart.findIndex(item => item.id === product.id);
  const updatedCart = existingIndex !== -1
    ? [...props.cart]
    : [...props.cart, { ...product, quantity: 1 }];

  emit('update-cart', updatedCart);
};

// Función para eliminar un producto
const removeProduct = async (product) => {
  const index = products.value.findIndex(item => item.id === product.id);
  if (index !== -1) {
    // Eliminar el producto de la lista de productos
    products.value.splice(index, 1);

    // Eliminar el producto del carrito si está presente
    const cartIndex = props.cart.findIndex(item => item.id === product.id);
    if (cartIndex !== -1) {
      props.cart.splice(cartIndex, 1);
      emit('update-cart', [...props.cart]); // Emitir una actualización del carrito
    }

    try {
      // Eliminar el producto en el servidor
      await deleteProductOnServer(product.id);
      // Mostrar mensaje de éxito
      successMessage.value = '¡Producto eliminado exitosamente!';
      setTimeout(() => {
        successMessage.value = ''; // Limpiar el mensaje después de unos segundos
      }, 3000);
    } catch (error) {
      console.error('Error al eliminar el producto en el servidor:', error);
    }
  }
};

// Función para eliminar un producto en el servidor
const deleteProductOnServer = async (productId) => {
  const response = await fetch(`${API}products/${productId}`, {
    method: 'DELETE'
  });
  if (!response.ok) {
    throw new Error('Error al eliminar el producto en el servidor');
  }
};

// Lista de productos filtrados y ordenados
const filteredProducts = computed(() => {
  return products.value.filter(product => {
    // Filtrar por texto de búsqueda y categoría seleccionada
    const matchesSearch = product.name.toLowerCase().includes(searchText.value.toLowerCase());
    const matchesCategory = categorySelected.value === '' || product.category === categorySelected.value;
    return matchesSearch && matchesCategory;
  }).sort((a, b) => {
    // Ordenar según las banderas de ordenamiento
    if (sortByName.value) {
      return (a.name < b.name) ? -1 : ((a.name > b.name) ? 1 : 0);
    } else if (sortByPrice.value) {
      return a.price - b.price;
    } else {
      return 0;
    }
  });
});
</script>

<style scoped>
/* Estilos específicos del componente */
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

  &:focus {
    outline: none;
    border-color: #999;
  }
}

.select-option {
  padding: 8px;
  font-size: 16px;
  color: #333;
  cursor: pointer;

  &:hover {
    background-color: #f2f2f2;
  }
}

.select-option:selected {
  background-color: #f2f2f2;
}

.sortable-header {
  cursor: pointer;
}

.sort-indicator {
  font-size: 12px;
  margin-left: 5px;
}

.success-message {
  background-color: #d4edda;
  color: #155724;
  border: 1px solid #c3e6cb;
  padding: 10px;
  margin-top: 10px;
  border-radius: 5px;
}

.alcoholic {
  background-color: #ffcccc;
  color: #333;
}

.product-table td:hover {
  background-color: #f2f2f2;
}
</style>
