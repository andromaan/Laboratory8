<template>
  <div class="container mx-auto p-4">
    <div class="flex justify-center">
      <div class="w-full max-w-4xl">
        <div class="rounded-md text-lg bg-gray-50 dark:bg-gray-400 dark:bg-opacity-10 text-black dark:text-white ring-1 ring-inset ring-gray-500 dark:ring-gray-400 ring-opacity-25 dark:ring-opacity-25">
          <div class="p-5">
              <h1 class="text-center mb-2">Create category</h1>
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

                <UButton class="flex mt-5" type="submit">
                  Create
                </UButton>
              </UForm>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { z } from 'zod';
import { reactive, ref, computed } from 'vue';
import axios from 'axios';

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

const categories = ref<Category[]>([]);

const getCategories = async () => {
  try {
    const response = await axios.get<Category[]>('http://127.0.0.1:8000/api/blog/categories');
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
  title: undefined,
  slug: undefined,
  description: undefined,
  parent_id: undefined
});

const onSubmit = async (event: Event) => {
  try {
    const result = schema.parse(state);
    console.log(result);

    const dataToSend = {
      ...result,
      parent_id: Number(result.parent_id)
    };

    const response = await axios.post('http://127.0.0.1:8000/api/blog/categories', dataToSend);
    alert("Category successfully created!");
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

<style scoped>
.container {
  max-width: 600px;
}
</style>
