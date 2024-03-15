<script setup>
import { computed, onMounted, provide, reactive, ref, watch } from 'vue';
import axios from 'axios';

import Header from '@/components/Header.vue';
import Drawer from '@/components/Drawer.vue';

import Home from './pages/Home.vue';

//ref() позволяет создавать реактивные переменные,
//которые можно использовать в шаблонах и других
//реактивных частях приложения.

const cart = ref([]);
const isCreatingOrder = ref(false);
//Структура данных: ref() используется для создания реактивной ссылки на одно значение, в то время как
//reactive() используется для создания реактивного объекта.

const drawerOpen = ref(false);

const totalPrice = computed(() => cart.value.reduce((acc, item) => acc + item.price, 0));
const vatPrice = computed(() => Math.round((totalPrice.value * 5) / 100));

const cartIsEmpty = computed(() => cart.value.length === 0);
const CartButtonDisabled = computed(() => isCreatingOrder.value || cartIsEmpty.value);

const closedDrawer = () => {
  drawerOpen.value = false;
};
const openDrawer = () => {
  drawerOpen.value = true;
};

const addToCart = (item) => {
  cart.value.push(item);
  item.isAdded = true;
};

const removeFromCart = (item) => {
  cart.value.splice(cart.value.indexOf(item), 1);
  item.isAdded = false;
};

const createOrder = async () => {
  try {
    isCreatingOrder.value = true;
    const { data } = await axios.post('https://5438d082aab2a032.mokky.dev/orders', {
      items: cart.value,
      totalPrice: totalPrice.value
    });

    cart.value = [];

    return data;
  } catch (err) {
    console.log(err);
  } finally {
    isCreatingOrder.value = false;
  }
};

//watch() в Vue.js — это функция из Composition API, которая
//позволяет отслеживать изменения реактивных ссылок

// тут watch отслеживает изменения в sortBy

watch(
  cart,
  () => {
    localStorage.setItem('cart', JSON.stringify(cart.value));
  },
  { deep: true }
);

provide('cart', {
  cart,
  closedDrawer,
  openDrawer,
  addToCart,
  removeFromCart
});
</script>

<template>
  <Drawer
    v-if="drawerOpen"
    :total-price="totalPrice"
    :vatPrice="vatPrice"
    @create-Order="createOrder"
    :CartButtonDisabled="CartButtonDisabled"
  />

  <div class="bg-white w-4/5 m-auto rounded-xl shadow-xl mt-14">
    <Header :total-price="totalPrice" @open-drawer="openDrawer" />

    <div class="p-10">
      <router-view> </router-view>
    </div>
  </div>
</template>

<style scoped></style>
