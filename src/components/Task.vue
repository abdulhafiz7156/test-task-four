<template>
  <div class="relative w-full p-4 bg-gray-50 rounded-lg shadow">
    <!-- Action Icons -->
    <div class="absolute top-2 right-2 flex space-x-2">
      <PencilSquareIcon
          @click="$emit('edit', task)"
          class="w-5 h-5 text-blue-500 cursor-pointer hover:text-blue-600"
      />
      <TrashIcon
          @click="$emit('delete', task.id)"
          class="w-5 h-5 text-red-500 cursor-pointer hover:text-red-600"
      />
    </div>

    <!-- Task Content -->
    <h3 class="text-lg font-semibold text-gray-800">{{ task.title }}</h3>
    <p class="text-gray-600">{{ task.body }}</p>
  </div>
</template>

<script>
import { TrashIcon, PencilSquareIcon } from '@heroicons/vue/24/outline';

export default {
  props: { task: Object },
  components: { TrashIcon, PencilSquareIcon },
};
</script>