<script setup lang="ts">
import { z } from 'zod';
import { reactive, ref, computed, onMounted } from 'vue';
import axios from 'axios';
import { useRoute } from 'vue-router';

interface Category {
  id: number;
  title: string;
  slug: string;
  description: string;
  created_at: string;
  updated_at: string;
  deleted_at: string;
  parent_id: number;
  parent_category: Category | null;
}

const route = useRoute();
const categoryId = ref<number | null>(null);
const category = ref<Category | null>(null);

const getCategory = async (id: number) => {
  try {
    const response = await axios.get<Category>(`http://127.0.0.1:8000/api/blog/categories/${id}`);
    category.value = response.data;
    state.title = category.value.title;
    state.slug = category.value.slug;
    state.description = category.value.description;
    state.parent_id = category.value.parent_id.toString();
  } catch (error) {
    console.error('Error fetching category:', error);
  }
};

onMounted(() => {
  const id = parseInt(route.params.id as string, 10);
  if (!isNaN(id)) {
    categoryId.value = id;
    getCategory(id);
  } else {
    console.error('Invalid Category ID');
  }
});

const categories = ref<Category[]>([]);
const getCategories = async () => {
  try {
    const response = await axios.get<Category[]>(`http://127.0.0.1:8000/api/blog/categories`);
    categories.value = response.data;
  } catch (error) {
    console.error('Error fetching categories:', error);
  }
};
getCategories();

const categoryOptions = computed(() => {
  return categories.value.map(category => ({
    value: category.id.toString(),
    label: category.title
  }));
});

const schema = z.object({
  title: z.string().min(5, 'Title must be at least 5 characters').max(200, 'Title must be less than 200 characters'),
  slug: z.string().max(200, 'Slug must be less than 200 characters').optional(),
  description: z.string().min(3, 'Description must be at least 3 characters').max(500, 'Description must be less than 500 characters').optional(),
  parent_id: z.string().refine(value => {
    const numberValue = Number(value);
    return Number.isInteger(numberValue) && categories.value.some(category => category.id === numberValue);
  }, 'Parent ID must be a valid integer and exist in categories')
});

type Schema = z.infer<typeof schema>;

const state = reactive<Partial<Schema>>({
  title: category.value?.title,
  slug: category.value?.slug,
  description: category.value?.description,
  parent_id: category.value?.parent_id?.toString()
});

const onSubmit = async (event: Event) => {
  try {
    const result = schema.parse(state);

    const dataToSend = {
      ...result,
      parent_id: Number(result.parent_id)
    };

    await axios.put(`http://127.0.0.1:8000/api/blog/categories/${categoryId.value}`, dataToSend);
    alert("Category successfully updated!");
    window.location.href = '/categories';
  } catch (e) {
    if (e instanceof z.ZodError) {
      console.error(e.errors);
    } else {
      console.error('Error submitting form:', e);
    }
  }
};
</script>
<template>
  <div class="container mx-auto p-4">
    <div class="flex justify-center">
      <div class="w-full max-w-4xl">
        <div class="rounded-md text-lg px-5 py-1 bg-gray-50 dark:bg-gray-400 dark:bg-opacity-10 text-black dark:text-white ring-1 ring-inset ring-gray-500 dark:ring-gray-400 ring-opacity-25 dark:ring-opacity-25">
          <div class="p-5">
            <div v-if="category" class="p-6">
              <div class="flex items-center justify-between mb-4">
                <p class="text-black dark:text-white"><strong>ID:</strong> {{ category.id }}</p>
              </div>
              <UForm :schema="schema" :state="state" class="space-y-4" @submit="onSubmit">
                <UFormGroup label="Title" name="title">
                  <UInput v-model="state.title" />
                </UFormGroup>
                <UFormGroup label="Slug" name="slug">
                  <UInput v-model="state.slug" />
                </UFormGroup>
                <UFormGroup label="Description" name="description">
                  <UInput v-model="state.description" />
                </UFormGroup>
                <UFormGroup label="Parent Category" name="parent_id">
                  <USelect v-model="state.parent_id" :options="categoryOptions" />
                </UFormGroup>
                <UButton type="submit">Save</UButton>
              </UForm>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<style scoped>
.container {
  max-width: 800px;
}
</style>
