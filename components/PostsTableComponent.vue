<script setup lang="ts">
useHead({
  title: 'PostsList'
});
import { ref, computed, watch } from 'vue';
import axios from 'axios';

const columns = [
  { key: 'id', label: '#', sortable: true },
  { key: 'author', label: 'Author', sortable: true },
  { key: 'category', label: 'Category', sortable: true },
  { key: 'title', label: 'Title', sortable: true },
  { key: 'publishedAt', label: 'Published At', sortable: true },
  { key: 'actions', label: 'Actions' }
];

const { data } = await useFetch<any>('http://127.0.0.1:8000/api/blog/posts');
const posts = ref(data.value);

const q = ref('');
const filteredRows = computed(() => {
  if (!q.value) {
    return posts.value;
  }

  return posts.value.filter((post) => {
    return Object.values(post).some((value) => {
      return String(value).toLowerCase().includes(q.value.toLowerCase());
    });
  });
});

const page = ref(1);
const pageCount = 7;

const sort = ref({ column: '', direction: 'asc' as const });
const sortedRows = computed(() => {
  const { column, direction } = sort.value;
  const sortedPosts = [...posts.value];

  if (column && direction) {
    sortedPosts.sort((a, b) => {
      const aValue = getColumnValue(a, column);
      const bValue = getColumnValue(b, column);

      if (aValue < bValue) return direction === 'asc' ? -1 : 1;
      if (aValue > bValue) return direction === 'asc' ? 1 : -1;
      return 0;
    });
  }

  return sortedPosts;
});

function getColumnValue(post: any, column: string) {
  switch (column) {
    case 'category':
      return post.category.title;
    case 'publishedAt':
      return post.published_at;
    default:
      return post[column];
  }
}

const rows = computed(() => {
  let filteredPosts = [...sortedRows.value];

  if (q.value) {
    filteredPosts = filteredPosts.filter(post => {
      return Object.values(post).some(value => {
        return String(value).toLowerCase().includes(q.value.toLowerCase());
      });
    });
  }

  const startIndex = (page.value - 1) * pageCount;
  const endIndex = startIndex + pageCount;

  return filteredPosts.slice(startIndex, endIndex);
});

watch(q, () => {
  page.value = 1;
});

const deletePost = (post : any) => {
  if (confirm(`Ви впевнені, що хочете видалити статтю '${post.title}'?`)) {
    axios.delete(`http://127.0.0.1:8000/api/blog/posts/${post.id}`)
        .then(res => {
          console.log(res);
          alert('Пост видалено!');
          posts.value = posts.value.filter(p => p.id !== post.id);
        })
        .catch(error => {
          console.error(error);
          alert('Не вдалось видалити пост');
        });
  }
};
</script>

<template>
  <title>Posts</title>
  <div>
    <div class="flex justify-around px-3 py-3.5 border-b border-gray-200 dark:border-gray-700">
      <UInput class="w-1/4" v-model="q" placeholder="Filter posts..." oninput="firstPage"/>
      <a href="posts/create"  class="inline-flex items-center font-medium rounded-md text-lg px-5 py-1 bg-green-400 text-white ring-1 ring-inset ring-green-400 ring-opacity-25 dark:ring-opacity-25">Додати</a>
    </div>
    <UTable :rows="rows" :columns="columns" v-model:sort="sort" sort-mode="manual">
      <template #author-data="{ row }">
        {{ row.user.name }}
      </template>
      <template #category-data="{ row }">
        {{ row.category.title }}
      </template>
      <template #title-data="{ row }">
        <a :href="'posts/' + row.id">{{ row.title }}</a>
      </template>
      <template #publishedAt-data="{ row }">
        {{ row.published_at ?? "N/A" }}
      </template>
      <template #actions-data="{ row }">
        <div class="flex gap-4">
          <a :href="'posts/update/' + row.id">Edit</a>
          <a href='#' @click="deletePost(row)">Delete</a>
        </div>

      </template>
    </UTable>

    <div class="flex justify-center px-3 py-3.5 border-t border-gray-200 dark:border-gray-700">
      <UPagination v-model="page" :page-count="pageCount" :total="filteredRows.length"/>
    </div>
  </div>
</template>
