<script setup>
import Card from '@/components/Card.vue';

defineProps({
  items: Array, //получаю прос из App с красовками,
  isFavorites: Boolean
});

const emit = defineEmits(['addToFavorite', 'addToCart']);
</script>

<template>
  <div class="grid grid-cols-4 gap-5" v-auto-animate>
    <Card
      v-for="item in items"
      :key="item.id"
      :id="item.id"
      :price="item.price"
      :imageUrl="item.imageUrl"
      :title="item.title"
      :isFavorite="item.isFavorite"
      :onclickAdd="isFavorites ? null : () => emit('addToCart', item)"
      :onClickFavorite="isFavorites ? null : () => emit('addToFavorite', item)"
      :isAdded="item.isAdded"
    />
  </div>
</template>
