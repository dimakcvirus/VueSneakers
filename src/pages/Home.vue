<script setup>
import { inject, reactive, watch, ref, onMounted } from 'vue';
import CardList from '../components/CardList.vue';
import axios from 'axios';

const { cart, addToCart, removeFromCart } = inject('cart');

const items = ref([]);
const filters = reactive({
  sortBy: 'title'
});

//тут от выбора в select мы передаем значение в sortBy
// а потом срабатывает watch
const onChangeSelect = (event) => {
  filters.sortBy = event.target.value;
};

const onClickAddPlus = (item) => {
  if (!item.isAdded) {
    addToCart(item);
  } else {
    removeFromCart(item);
  }
  console.log(cart.value);
};

const onChangeSearchInput = (event) => {
  filters.searchQuery = event.target.value;
};

const addToFavorite = async (item) => {
  try {
    if (!item.isFavorite) {
      const obj = {
        item_id: item.id
      };
      item.isFavorite = true;
      const { data } = await axios.post(`https://5438d082aab2a032.mokky.dev/favorites`, obj);
      item.favoriteId = data.id;
    } else {
      item.isFavorite = false;
      await axios.delete(`https://5438d082aab2a032.mokky.dev/favorites/${item.favoriteId}`);
      item.favoriteId = null;
    }
  } catch (err) {
    console.log(err);
  }
};

const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get(`https://5438d082aab2a032.mokky.dev/favorites`);
    items.value = items.value.map((item) => {
      const favorite = favorites.find((favorite) => favorite.item_id === item.id);
      if (!favorite) {
        return item;
      }
      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id
      };
    });
  } catch (err) {
    console.log(err);
  }
};

const fetchItems = async () => {
  // -----через обычный фетч с зенми ----------

  // fetch('https://5438d082aab2a032.mokky.dev/items')
  //   .then((res) => res.json()) //сокращеная стрелачная функция котора сразу возвращает результат
  //   .then((data) => {
  //     console.log(data);
  //   });

  // -----через обычный фетч с зенми ----------

  // -----через аксиос с авейтом ----------
  try {
    const params = {
      sortBy: filters.sortBy
    };

    if (filters.searchQuery) {
      params.title = `*${filters.searchQuery}*`;
    }

    const { data } = await axios.get(`https://5438d082aab2a032.mokky.dev/items`, {
      params
    });
    items.value = data.map((obj) => ({
      ...obj,
      isFavorite: false,
      favoriteId: null,
      isAdded: false
    }));
  } catch (err) {
    console.log(err);
  }

  // -----через аксиос с авейтом ----------

  // -----через аксиос без авейтом ----------

  // axios.get('https://5438d082aab2a032.mokky.dev/items').then((resp) => console.log(resp.data));

  // -----через аксиос без авейтом ----------

  // getURL();
  // -----обычный асинк эвейт ----------
  // const getURL = async () => {
  //   const response = await fetch('https://5438d082aab2a032.mokky.dev/items');
  //   const data = await response.json();
  //   console.log(data);
  // };
};

// для первой отрисовки есть специальный хук в vue
// называется он onMounted() принимает в себя колбэк функцию
onMounted(async () => {
  const localCart = localStorage.getItem('cart');
  cart.value = localCart ? JSON.parse(localCart) : [];

  await fetchItems();
  await fetchFavorites();

  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cart.value.some((cartItem) => cartItem.id === item.id)
  }));
});

watch(cart, () => {
  items.value = items.value.map((item) => ({
    ...item,
    isAdded: false
  }));
});

watch(filters, fetchItems);
</script>

<template>
  <div class="flex justify-between items-center">
    <h2 class="text-3xl font-bold mb-8">Все кросовки</h2>

    <div class="flex gap-4">
      <select @change="onChangeSelect" class="py-2 px-3 border rounded-md outline-none bg-white">
        <option value="name">По названию</option>
        <option value="price">По цене (дешевые)</option>
        <option value="-price">По цене (Дорогие)</option>
      </select>

      <div class="relative">
        <img class="absolute left-4 top-3" src="/public/search.svg" alt="" />
        <input
          @input="onChangeSearchInput"
          class="border rounded-md py-2 pl-11 pr-4 outline-none focus:border-gray-400"
          placeholder="Поиск..."
        />
      </div>
    </div>
  </div>
  <div class="mt-10">
    <CardList :items="items" @addToFavorite="addToFavorite" @add-to-cart="onClickAddPlus" />
  </div>
</template>
