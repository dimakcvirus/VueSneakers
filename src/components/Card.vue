<script setup>
const props = defineProps({
  id: Number,
  imageUrl: String,
  title: String,
  price: Number,
  isFavorite: Boolean,
  isAdded: Boolean,
  onClickFavorite: Function, //передача функции
  onclickAdd: Function
});
</script>
<template>
  <div
    class="relative bg-white border-slate-100 rounded-3xl p-8 cursor-pointer transition hover:-translate-y-2 hover:shadow-xl"
  >
    <img
      v-if="onClickFavorite"
      :src="!isFavorite ? '/like-1.svg' : '/like-2.svg'"
      alt="like 1"
      class="absolute top-8 left-8"
      @click="onClickFavorite"
    />
    <!-- принимается пропс imageUrl и отрисовывается изображение -->
    <img :src="imageUrl" alt="Sneaker" />
    <p class="mt-2">{{ title }}</p>
    <!-- а так же прайс  наименование  -->
    <div class="flex justify-between">
      <div class="flex flex-col">
        <span class="text-slate-4 00">Цена:</span>
        <span>{{ price }} руб.</span>
      </div>

      <img
        v-if="onClickFavorite"
        @click="onclickAdd"
        :src="!isAdded ? '/plus.svg' : '/checked.svg'"
        alt="plus"
      />
    </div>
  </div>
</template>
