<template>
  <div v-if="successMessage" class="success-message">
    {{ successMessage }}
  </div>
  <div>
    <input type="text" v-model="searchText" placeholder="Buscar por nombre" class="search-input">
    <div class="select-container">
      <label class="select-label">Categoría:</label>
      <select v-model="categorySelected" class="select-input">
        <option value="" class="select-option">Todas</option>
        <option v-for="category in categories" :key="category" :value="category" class="select-option">{{ category }}
        </option>
      </select>
    </div>
    <table class="product-table">
      <thead>
        <tr>
          <th>ID</th>
          <th>
            <span @click="sortByName = !sortByName" class="sortable-header">Nombre</span>
            <span v-if="sortByName.value" class="sort-indicator">&#8593;</span>
            <span v-else class="sort-indicator">&#8595;</span>
          </th>
          <th>
            <span @click="sortByPrice = !sortByPrice" class="sortable-header">Precio</span>
            <span v-if="sortByPrice.value" class="sort-indicator">&#8593;</span>
            <span v-else class="sort-indicator">&#8595;</span>
          </th>
          <th>Categoría</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="product in filteredProducts" :key="product.id" :class="{ alcoholic: product.category == 'Alcoholic' }">
          <td>{{ product.id }}</td>
          <td>{{ product.name }}</td>
          <td>{{ product.price }}</td>
          <td>{{ product.category }}</td>
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
import { ref, onMounted, computed } from 'vue';

const API = "http://localhost:3000/api/";

const emit = defineEmits(['product-added', 'update-cart']);

const products = ref([]);
const searchText = ref('');
const categories = ref([]);
const categorySelected = ref('');
const sortByName = ref(false);
const sortByPrice = ref(false);
const props = defineProps({
  cart: Array
});
const successMessage = ref('');


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

onMounted(fetchProducts);
onMounted(fetchCategories);

const getProducts = async () => {
  const response = await fetch(API + 'products');
  return response.json();
};

const getCategories = async () => {
  const response = await fetch(API + 'categories');
  return response.json();
};

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
      await deleteProductOnServer(product.id);
      successMessage.value = '¡Producto eliminado exitosamente!';
      setTimeout(() => {
        successMessage.value = ''; // Limpiar el mensaje después de unos segundos
      }, 3000);
    } catch (error) {
      console.error('Error al eliminar el producto en el servidor:', error);
    }
  }
};

const deleteProductOnServer = async (productId) => {
  const response = await fetch(`${API}products/${productId}`, {
    method: 'DELETE'
  });
  if (!response.ok) {
    throw new Error('Error al eliminar el producto en el servidor');
  }
};

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

</script>

<style scoped>
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
