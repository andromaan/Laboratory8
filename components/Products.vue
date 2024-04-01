<script setup lang="ts">
useHead({
  title: 'ProductsList'
});
import { ref, computed, watch } from 'vue';

const columns = [
  { key: 'title', label: 'Title', sortable: true },
  { key: 'description', label: 'Description', sortable: true },
  { key: 'price', label: 'Price', sortable: true },
  { key: 'rating', label: 'Rating', sortable: true },
  { key: 'brand', label: 'Brand', sortable: true },
  { key: 'category', label: 'Category', sortable: true },
  { key: 'thumbnail', label: 'Thumbnail' }
];

const { data } = await useFetch<any>('https://dummyjson.com/products');

const products = ref(data.value.products);

const q = ref('');
const filteredRows = computed(() => {
  if (!q.value) {
    return products.value;
  }

  return products.value.filter((product) => {
    return Object.values(product).some((value) => {
      return String(value).toLowerCase().includes(q.value.toLowerCase());
    });
  });
});

const page = ref(1);
const pageCount = 3;

const sort = ref({ column: '', direction: 'asc' as const })
const sortedRows = computed(() => {
  const sortedProducts = [...products.value]
  const { column, direction } = sort.value

  if (column && direction) {
    sortedProducts.sort((a, b) => {
      const aValue = a[column]
      const bValue = b[column]
      if (aValue < bValue) return direction === 'asc' ? -1 : 1
      if (aValue > bValue) return direction === 'asc' ? 1 : -1
      return 0
    })
  }

  return sortedProducts
})

const rows = computed(() => {
  let filteredProducts = [...sortedRows.value]

  if (q.value) {
    filteredProducts = filteredProducts.filter(product => {
      return Object.values(product).some(value => {
        return String(value).toLowerCase().includes(q.value.toLowerCase())
      })
    })
  }

  const startIndex = (page.value - 1) * pageCount
  const endIndex = startIndex + pageCount

  return filteredProducts.slice(startIndex, endIndex)
})

watch(q, ()=>{
  page.value = 1;
});

</script>

<template>
  <title>Products</title>
  <div>
    <div class="flex justify-center px-3 py-3.5 border-b border-gray-200 dark:border-gray-700">
      <UInput class="w-1/4" v-model="q" placeholder="Filter product..." oninput="firstPage"/>
    </div>
    <UTable :rows="rows" :columns="columns" v-model:sort="sort" sort-mode="manual">
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
