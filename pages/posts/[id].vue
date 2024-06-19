<template>
  <div class="container mx-auto p-4">
    <div class="flex justify-center">
      <div class="w-full max-w-4xl">
        <div class="rounded-md text-lg px-5 py-1 bg-gray-50 dark:bg-gray-400 dark:bg-opacity-10 text-black dark:text-white ring-1 ring-inset ring-gray-500 dark:ring-gray-400 ring-opacity-25 dark:ring-opacity-25">
          <div v-if="post" class="p-6">
            <div class="flex items-center justify-between mb-4">
              <h1 class="text-2xl font-bold">{{ post.title }}</h1>
              <p class="text-black dark:text-white"><strong>ID:</strong> {{ post.id }}</p>
            </div>
            <div class="mb-4">
              <p class="text-black dark:text-white"><strong>Автор:</strong> {{ post.user.name }}</p>
              <p class="text-black dark:text-white"><strong>Категорія:</strong> {{ post.category.title }}</p>
              <p class="text-black dark:text-white"><strong>Дата публікації:</strong> {{ post.published_at }}</p>
              <p class="text-black dark:text-white"><strong>Опубліковано:</strong> {{ post.is_published ? "Так" : "Ні" }}</p>
            </div>
            <div class="prose" v-html="post.content"></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue';
import { useRoute } from 'vue-router';

const post = ref(null);
const route = useRoute();

const getPostDetails = async (id) => {
  try {
    const response = await fetch(`http://127.0.0.1:8000/api/blog/posts/${id}`);
    if (response.ok) {
      post.value = await response.json();
    } else {
      throw new Error('Невдалося вивести данні');
    }
  } catch (error) {
    console.error('Помилка: ', error);
  }
};

onMounted(() => {
  if (route.params.id) {
    getPostDetails(route.params.id);
  }
});

watch(() => route.params.id, (newId) => {
  if (newId) {
    getPostDetails(newId);
  }
});
</script>

<style scoped>
.prose {
  max-width: none;
}
</style>
