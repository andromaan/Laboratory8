  <script setup lang="ts">
  useHead({
    title: 'CategoriesList'
  });
  import { ref, computed, watch } from 'vue';

  const columns = [
    {key: 'id', label: '#'},
    {key: 'title', label: 'Назва'},
    {key: 'slug', label: 'Силка'},
    {key: 'description', label: 'Опис'},
    {key: 'created_at', label: 'Дата створення'},
    {key: 'updated_at', label: 'Дата оновлення'},
    {key: 'deleted_at', label: 'Дата видалення'},
    {key: 'parent_category.title', label: 'Батьківська категорія'},
    {key: 'actions', label: 'Actions'}
  ];

  const { data } = await useFetch<any>('http://127.0.0.1:8000/api/blog/categories');
  const categories = ref(data.value);

  const q = ref('');
  const filteredCategories = computed(() => {
    if (!q.value) {
      return categories.value;
    }

    return categories.value.filter((category) => {
      return Object.values(category).some((value) => {
        return String(value).toLowerCase().includes(q.value.toLowerCase());
      });
    });
  });

  const page = ref(1);
  const pageCount = 7;

  const sort = ref({ column: '', direction: 'asc' as const });
  const sortedCategories = computed(() => {
    const { column, direction } = sort.value;
    const sortedCategories = [...categories.value];

    if (column && direction) {
      sortedCategories.sort((a, b) => {
        const aValue = getColumnValue(a, column);
        const bValue = getColumnValue(b, column);

        if (aValue < bValue) return direction === 'asc' ? -1 : 1;
        if (aValue > bValue) return direction === 'asc' ? 1 : -1;
        return 0;
      });
    }

    return sortedCategories;
  });

  function getColumnValue(category: any, column: string) {
    switch (column) {
      case 'parent_title':
        return category.parent_category || '';
      default:
        return category[column];
    }
  }

  const rows = computed(() => {
    let filteredCategoriesList = [...sortedCategories.value];

    if (q.value) {
      filteredCategoriesList = filteredCategoriesList.filter(category => {
        return Object.values(category).some(value => {
          return String(value).toLowerCase().includes(q.value.toLowerCase());
        });
      });
    }

    const startIndex = (page.value - 1) * pageCount;
    const endIndex = startIndex + pageCount;

    return filteredCategoriesList.slice(startIndex, endIndex);
  });

  watch(q, () => {
    page.value = 1;
  });
  </script>

  <template>
    <title>Categories</title>
    <div>
      <div class="flex justify-around px-3 py-3.5 border-b border-gray-200 dark:border-gray-700">
        <UInput class="w-1/4" v-model="q" placeholder="Filter categories..." oninput="firstPage"/>
        <a href="categories/create" class="inline-flex items-center font-medium rounded-md text-lg px-5 py-1 bg-green-400 text-white ring-1 ring-inset ring-green-400 ring-opacity-25 dark:ring-opacity-25">Додати</a>
      </div>
      <UTable :rows="rows" :columns="columns" v-model:sort="sort" sort-mode="manual">
        <template #title-data="{ row }">
          <a :href="'categories/' + row.id">{{ row.title ?? 'N/A' }}</a>
        </template>
        <template #parent_category.title-data="{ row }">
          {{ row.parent_category?.title ?? 'N/A' }}
        </template>
        <template #description-data="{ row }">
          {{ row.description ?? 'N/A' }}
        </template>
        <template #created_at-data="{ row }">
          {{ row.created_at ?? 'N/A' }}
        </template>
        <template #updated_at-data="{ row }">
          {{ row.updated_at ?? 'N/A' }}
        </template>
        <template #deleted_at-data="{ row }">
          {{ row.deleted_at ?? 'N/A' }}
        </template>

        <template #actions-data="{ row }">
          <a :href="'categories/update/' + row.id">Edit</a>
        </template>
      </UTable>

      <div class="flex justify-center px-3 py-3.5 border-t border-gray-200 dark:border-gray-700">
        <UPagination v-model="page" :page-count="pageCount" :total="filteredCategories.length"/>
      </div>
    </div>
  </template>
