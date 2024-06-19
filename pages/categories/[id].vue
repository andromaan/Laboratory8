<template>
  <div class="container mx-auto p-4">
    <div class="flex justify-center">
      <div class="w-full max-w-4xl">
        <div class="rounded-md text-lg px-5 py-1 bg-gray-50 dark:bg-gray-400 dark:bg-opacity-10 text-black dark:text-white ring-1 ring-inset ring-gray-500 dark:ring-gray-400 ring-opacity-25 dark:ring-opacity-25">
          <div v-if="category" class="p-6">
            <div class="flex items-center justify-between mb-4">
              <h1 class="text-2xl font-bold">{{ category.title ?? "N/A" }}</h1>
              <p class="text-black dark:text-white"><strong>ID:</strong> {{ category.id }}</p>
            </div>
            <div class="mb-4">
              <p class="text-black dark:text-white"><strong>Силка:</strong> {{ category.slug ?? "N/A"}}</p>
              <p class="text-black dark:text-white"><strong>Опис:</strong> {{ category.description ?? "N/A"}}</p>
              <p class="text-black dark:text-white"><strong>Дата створення:</strong> {{ category.created_at ?? "N/A"}}</p>
              <p class="text-black dark:text-white"><strong>Дата оновлення:</strong> {{ category.updated_at ?? "N/A"}}</p>
              <p class="text-black dark:text-white"><strong>Дата видалення:</strong> {{ category.deleted_at ?? "N/A"}}</p>
              <p class="text-black dark:text-white"><strong>Батьківська категорія:</strong> {{ category.parent_category?.title || 'N/A' }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue';
import { useRoute } from 'vue-router';

const category = ref(null);
const route = useRoute();

const getCategoryDetails = async (id) => {
  try {
    const response = await fetch(`http://127.0.0.1:8000/api/blog/categories/${id}`);
    if (response.ok) {
      category.value = await response.json();
    } else {
      throw new Error('Не вдалося вивести дані');
    }
  } catch (error) {
    console.error('Помилка: ', error);
  }
};

onMounted(() => {
  if (route.params.id) {
    getCategoryDetails(route.params.id);
  }
});

watch(() => route.params.id, (newId) => {
  if (newId) {
    getCategoryDetails(newId);
  }
});
</script>

<style scoped>
.prose {
  max-width: none;
}
</style>
