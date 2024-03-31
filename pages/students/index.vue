<script lang="ts">
import { defineComponent } from 'vue';
import StudentList from "~/components/StudentsList.vue";
export default defineComponent({
  name: 'Student',
  components: StudentList
});
</script>

<template>
  <StudentsList/>
</template>



<style scoped>

</style>