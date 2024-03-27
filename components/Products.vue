<script setup lang="ts">
import { ref, computed } from 'vue';

interface Product {
  id: number;
  title: string;
  description: string;
  price: number;
  rating: number;
  brand: string;
  category: string;
  thumbnail: string;
}

const columns = [
  { key: 'title', label: 'Title' },
  { key: 'description', label: 'Description' },
  { key: 'price', label: 'Price' },
  { key: 'rating', label: 'Rating' },
  { key: 'brand', label: 'Brand' },
  { key: 'category', label: 'Category' },
  { key: 'thumbnail', label: 'Thumbnail' }
];

const { data } = await useFetch<any>('https://dummyjson.com/products');
const products: Product[] = data.value.products;

const q = ref('');
const filteredRows = computed(() => {
  if (!q.value) {
    return products;
  }

  return products.filter((product: Product) => { // Вказуємо тип параметра явно
    return Object.values(product).some((value) => {
      return String(value).toLowerCase().includes(q.value.toLowerCase());
    });
  });
});

const page = ref(1);
const pageCount = 3;

const rows = computed(() => {
  return filteredRows.value.slice((page.value - 1) * pageCount, page.value * pageCount);
});
</script>

<template>
  <div>
    <div class="flex justify-center px-3 py-3.5 border-b border-gray-200 dark:border-gray-700">
      <UInput class="w-1/4" v-model="q" placeholder="Filter product..."/>
    </div>
    <UTable :rows="rows" :columns="columns" >
      <template #thumbnail-data="{row}">
          <img :src="row.thumbnail" alt="" class="h-[100px] w-[100px]">
      </template>
      <template #description-data="{row}">
        <div class="text-wrap">{{ row.description }}</div>
      </template>
      <template #rating-data="{row}">
        <div :class="{'inline-flex items-center font-medium rounded-md text-xs px-1.5 py-0.5 bg-red-50 dark:bg-red-400 dark:bg-opacity-10 text-red-500 dark:text-red-400 ring-1 ring-inset ring-red-500 dark:ring-red-400 ring-opacity-25 dark:ring-opacity-25':
        row.rating < 4.5,
        'inline-flex items-center font-medium rounded-md text-xs px-1.5 py-0.5 bg-emerald-50 dark:bg-emerald-400 dark:bg-opacity-10 text-emerald-500 dark:text-emerald-400 ring-1 ring-inset ring-emerald-500 dark:ring-emerald-400 ring-opacity-25 dark:ring-opacity-25':
        row.rating >= 4.5}">
          {{ row.rating }}
        </div>
      </template>
    </UTable>

    <div class="flex justify-center px-3 py-3.5 border-t border-gray-200 dark:border-gray-700">
      <UPagination v-model="page" :page-count="pageCount" :total="filteredRows.length"/>
    </div>
  </div>
</template>