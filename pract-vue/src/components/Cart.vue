<template>
    <div class="cart">
        <button @click="goBackToProducts">Volver a la lista de productos</button>
        <h2>Carrito de Compras</h2>
        <p v-if="!cart.length">No hay productos en el carrito.</p>
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
                <tr v-for="(item, index) in cart" :key="index">
                    <td style="width: 200px;">{{ item.name }}</td>
                    <td style="width: 100px;">{{ item.price }}</td>
                    <td class="quantity" style="width: 150px;">
                        <button @click="decrementQuantity(index)">-</button>
                        <input type="number" v-model.number="item.quantity" min="0" @blur="checkQuantity(index)"
                            @input="checkInput(index)" />
                        <button @click="incrementQuantity(index)">+</button>
                    </td>
                    <td style="width: 100px;">{{ item.price * item.quantity }}</td>
                    <td class="actions" style="width: 150px;">
                        <button @click="removeFromCart(index)">Eliminar</button>
                    </td>
                </tr>
            </tbody>
        </table>
        <div class="cart-total">
            <p>Total del Pedido: {{ total }}</p>
            <button @click="emptyCart">Vaciar Carrito</button>
            <button @click="clearLocalStorage">Eliminar Carrito Almacenado</button>
        </div>
    </div>
</template>
  
<script setup>
import { computed, defineProps, defineEmits, onMounted } from 'vue';

const emit = defineEmits(['back-to-products', 'update-cart']);

const props = defineProps({
    cart: {
        type: Array,
        default: () => [],
    },
});

const total = computed(() => {
    return props.cart.reduce((acc, item) => acc + (item.price * item.quantity), 0);
});

const removeFromCart = (index) => {
    props.cart.splice(index, 1);
    saveCartToLocalStorage();
};

const emptyCart = () => {
    props.cart = [];
    saveCartToLocalStorage();
};

const goBackToProducts = () => {
    emit('back-to-products');
};

const decrementQuantity = (index) => {
    if (props.cart[index].quantity > 0) {
        props.cart[index].quantity--;
        if (props.cart[index].quantity === 0) {
            removeFromCart(index);
        }
        saveCartToLocalStorage();
    }
};

const incrementQuantity = (index) => {
    props.cart[index].quantity++;
    saveCartToLocalStorage();
};

const checkQuantity = (index) => {
    if (props.cart.length > 0 && props.cart[index].quantity <= 0) {
        removeFromCart(index);
    }
};

const checkInput = (index) => {
    if (props.cart.length > 0 && props.cart[index].quantity <= 0) {
        removeFromCart(index);
    }
};

const saveCartToLocalStorage = () => {
    try {
        const serializedCart = JSON.stringify(props.cart);
        localStorage.setItem('cart', serializedCart);
    } catch (error) {
        console.error('Error saving cart to local storage:', error);
    }
};


const clearLocalStorage = () => {
    localStorage.removeItem('cart');
};

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


const handleUpdateCart = (updatedCart) => {
    props.cart = updatedCart;
    saveCartToLocalStorage(); // Update local storage as well
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
  