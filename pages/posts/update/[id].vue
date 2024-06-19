<template>
  <div class="container mx-auto p-4">
    <div class="flex justify-center">
      <div class="w-full max-w-4xl">
        <div class="rounded-md text-lg px-5 py-1 bg-gray-50 dark:bg-gray-400 dark:bg-opacity-10 text-black dark:text-white ring-1 ring-inset ring-gray-500 dark:ring-gray-400 ring-opacity-25 dark:ring-opacity-25">
          <div v-if="post" class="p-6">
            <h1 class="text-2xl font-bold mb-4">Editing post {{ post.title }}</h1>
            <UForm v-if="categoryOptions.length > 0" :schema="schema" :state="state" @submit="onSubmit">
              <UFormGroup label="Title" name="title">
                <UInput v-model="state.title" />
              </UFormGroup>
              <UFormGroup label="Slug" name="slug">
                <UInput v-model="state.slug" />
              </UFormGroup>
              <UFormGroup label="Content" name="content_raw">
                <UTextarea v-model="state.content_raw" />
              </UFormGroup>
              <UFormGroup label="Excerpt" name="excerpt">
                <UInput v-model="state.excerpt" />
              </UFormGroup>
              <UFormGroup label="Category" name="category_id">
                <USelect v-model="state.category_id" :options="categoryOptions" />
              </UFormGroup>
              <UFormGroup label="Is published" name="is_published">
                <UCheckbox v-model="state.is_published" />
              </UFormGroup>
              <UButton type="submit" class="my-3">Save</UButton>
            </UForm>
            <div v-else>
              Loading categories...
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { z } from 'zod';
import { reactive, ref, computed, onMounted } from 'vue';
import axios from 'axios';
import { useRoute } from 'vue-router';

interface Category {
  id: number;
  title: string;
}

interface Post {
  id: number;
  title: string;
  slug: string;
  content_raw: string;
  excerpt: string;
  category_id: number;
  is_published: boolean;
}

const route = useRoute();
const postId = ref<number | null>(null);
const post = ref<Post | null>(null);

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

const getPost = async (id: number) => {
  try {
    const response = await axios.get<Post>(`http://127.0.0.1:8000/api/blog/posts/${id}`);
    post.value = response.data;
    state.title = post.value.title;
    state.slug = post.value.slug;
    state.content_raw = post.value.content_raw;
    state.excerpt = post.value.excerpt;
    state.category_id = post.value.category_id.toString();
    state.is_published = post.value.is_published;
  } catch (error) {
    console.error('Error fetching post:', error);
  }
};

onMounted(() => {
  const id = parseInt(route.params.id as string, 10);
  if (!isNaN(id)) {
    postId.value = id;
    getPost(id);
  } else {
    console.error('Invalid post ID');
  }
});

const schema = z.object({
  title: z.string().min(5, 'Title must be at least 5 characters').max(200, 'Title must be less than 200 characters'),
  slug: z.string().max(200, 'Slug must be less than 200 characters').optional(),
  content_raw: z.string().min(5, 'Content must be at least 5 characters').max(10000, 'Content must be less than 10000 characters'),
  excerpt: z.string().optional(),
  category_id: z.string().refine(value => {
    const numberValue = Number(value);
    return Number.isInteger(numberValue) && categories.value.some(category => category.id === numberValue);
  }, 'Category must be a valid selection'),
  is_published: z.boolean().optional()
});

type Schema = z.infer<typeof schema>;

const state = reactive<Partial<Schema>>({
  title: undefined,
  slug: undefined,
  content_raw: undefined,
  excerpt: undefined,
  category_id: undefined,
  is_published: undefined
});

const onSubmit = async (event: Event) => {
  try {
    const result = schema.parse(state);
    const dataToSend = {
      ...result,
      category_id: Number(result.category_id)
    };

    await axios.put(`http://127.0.0.1:8000/api/blog/posts/${postId.value}`, dataToSend);
    alert('Post successfully updated!');
    window.location.href = '/posts';
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
  max-width: 800px;
}
</style>
