<script setup lang="ts">
useHead({
  title: 'StudentsList'
});

const columns = [
  { key: 'id', label: 'Id' },
  { key: 'name', label: 'Name', sortable: true },
  { key: 'email', label: 'Email', sortable: true },
  { key: 'group', label: 'Group', sortable: true }
];

const students = [
  {
    id: 1,
    name: 'Лавренюк Андрій',
    email: 'andrii.lavreniuk@oa.edu.ua',
    group: 'КН-1'
  },
  {
    id: 2,
    name: 'Іваненко Олександр',
    email: 'oleksandr.ivanenko@oa.edu.ua',
    group: 'КН-2'
  },
  {
    id: 3,
    name: 'Петренко Марія',
    email: 'maria.petrenko@oa.edu.ua',
    group: 'КН-1'
  },
  {
    id: 4,
    name: 'Ковальчук Ірина',
    email: 'iryna.kovalchuk@oa.edu.ua',
    group: 'КН-3'
  },
  {
    id: 5,
    name: 'Григоренко Олег',
    email: 'oleg.grigorenko@oa.edu.ua',
    group: 'КН-2'
  },
  {
    id: 6,
    name: 'Сидоренко Наталія',
    email: 'nataliya.sydorenko@oa.edu.ua',
    group: 'КН-1'
  },
  {
    id: 7,
    name: 'Мельник Юрій',
    email: 'yurii.melnyk@oa.edu.ua',
    group: 'КН-3'
  },
  {
    id: 8,
    name: 'Олійник Віктор',
    email: 'viktor.oliynyk@oa.edu.ua',
    group: 'КН-2'
  },
  {
    id: 9,
    name: 'Бойко Тетяна',
    email: 'tetiana.boiko@oa.edu.ua',
    group: 'КН-1'
  },
  {
    id: 10,
    name: 'Марченко Василь',
    email: 'vasyl.marchenko@oa.edu.ua',
    group: 'КН-3'
  },
  {
    id: 11,
    name: 'Соловйова Оксана',
    email: 'oksana.soloviova@oa.edu.ua',
    group: 'КН-2'
  },
  {
    id: 12,
    name: 'Павленко Сергій',
    email: 'serhii.pavlenko@oa.edu.ua',
    group: 'КН-1'
  },
  {
    id: 13,
    name: 'Мироненко Ірина',
    email: 'iryna.myronenko@oa.edu.ua',
    group: 'КН-3'
  },
  {
    id: 14,
    name: 'Кузьменко Олена',
    email: 'olena.kuzmenko@oa.edu.ua',
    group: 'КН-2'
  },
  {
    id: 15,
    name: 'Даниленко Вікторія',
    email: 'viktoriia.danilenko@oa.edu.ua',
    group: 'КН-1'
  },
  {
    id: 16,
    name: 'Мосійчук Олександр',
    email: 'oleksandr.mosiichuk@oa.edu.ua',
    group: 'КН-2'
  },
];

const sort = ref({ column: 'title', direction: 'asc' as const })
const sortedRows = computed(() => {
  const sortedProducts = [...students]
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

const page = ref(1);
const pageCount = 4;

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

const q = ref('');
const filteredRows = computed(() => {
  if (!q.value) {
    return students
  }

  return students.filter((person) => {
    return Object.values(person).some((value) => {
      return String(value).toLowerCase().includes(q.value.toLowerCase())
    })
  })
});


watch(q, () => {
  page.value = 1
});

</script>

<template>
  <div>
    <div class="flex justify-center px-3 py-3.5 border-b border-gray-200 dark:border-gray-700">
      <UInput class="w-1/4" v-model="q" placeholder="Filter people..."/>
    </div>

    <UTable :rows="rows" , :columns="columns" v-model:sort="sort"/>

    <div class="flex justify-center px-3 py-3.5 border-t border-gray-200 dark:border-gray-700">
      <UPagination v-model="page" :page-count="pageCount" :total="filteredRows.length"/>
    </div>
  </div>
</template>