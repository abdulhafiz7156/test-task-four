<template>
  <div class="min-h-screen bg-gray-100 flex justify-center p-4">
    <div class="w-full max-w-4xl bg-white p-6 rounded-lg shadow-md">
      <!-- Header -->
      <div class="flex justify-between items-center mb-6">
        <h1 class="text-2xl font-bold text-gray-800">Task Manager</h1>
        <button
            class="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600"
            @click="openCreateModal"
        >
          Add Task
        </button>
      </div>

      <!-- Task List -->
      <div v-if="tasks.length" class="space-y-4">
        <Task
            v-for="task in tasks"
            :key="task.id"
            :task="task"
            @delete="deleteTask"
            @edit="openEditModal"
        />
      </div>
      <p v-else class="text-gray-600 text-center">No tasks available. Add some!</p>
    </div>
  </div>

  <!-- Task Modal -->
  <div
      v-if="isModalOpen"
      class="fixed inset-0 bg-black bg-opacity-50 flex justify-center items-center"
  >
    <div class="bg-white p-6 rounded-lg shadow-lg w-96">
      <h2 class="text-xl font-bold mb-4">
        {{ isEditing ? 'Edit Task' : 'Create New Task' }}
      </h2>

      <!-- Title Input -->
      <input
          v-model="modalTask.title"
          type="text"
          placeholder="Task Title"
          class="w-full p-2 border border-gray-300 rounded mb-1"
      />
      <p v-if="errors.title" class="text-red-500 text-sm mb-2">{{ errors.title }}</p>

      <!-- Description Input -->
      <textarea
          v-model="modalTask.body"
          placeholder="Task Description"
          class="w-full p-2 border border-gray-300 rounded mb-1"
      ></textarea>
      <p v-if="errors.body" class="text-red-500 text-sm mb-4">{{ errors.body }}</p>

      <!-- Buttons -->
      <div class="flex justify-end space-x-2">
        <button
            class="px-4 py-2 bg-gray-300 rounded hover:bg-gray-400"
            @click="closeModal"
        >
          Cancel
        </button>
        <button
            class="px-4 py-2 text-white rounded"
            :class="isEditing ? 'bg-yellow-500 hover:bg-yellow-600' : 'bg-green-500 hover:bg-green-600'"
            @click="isEditing ? updateTask() : createTask()"
        >
          {{ isEditing ? 'Save Changes' : 'Create' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import Task from './components/Task.vue';

export default {
  components: { Task },
  setup() {
    const tasks = ref([]);
    const isModalOpen = ref(false);
    const isEditing = ref(false);
    const modalTask = ref({ id: null, title: '', body: '' });
    const errors = ref({ title: '', body: '' });

    const TASKS_KEY = 'vue-tasks';

    // 📥 Load tasks from localStorage or fallback to API
    const loadTasks = async () => {
      const storedTasks = localStorage.getItem(TASKS_KEY);

      if (storedTasks) {
        tasks.value = JSON.parse(storedTasks);
      } else {
        const res = await fetch('https://jsonplaceholder.typicode.com/posts?_limit=10');
        tasks.value = await res.json();
        saveTasks(); // Save fetched tasks to localStorage
      }
    };

    // 💾 Save tasks to localStorage
    const saveTasks = () => {
      localStorage.setItem(TASKS_KEY, JSON.stringify(tasks.value));
    };

    onMounted(loadTasks);

    const validateForm = () => {
      errors.value = { title: '', body: '' };
      let isValid = true;

      if (!modalTask.value.title.trim()) {
        errors.value.title = 'Title cannot be empty';
        isValid = false;
      }

      if (!modalTask.value.body.trim()) {
        errors.value.body = 'Description cannot be empty';
        isValid = false;
      }

      return isValid;
    };

    const openCreateModal = () => {
      isEditing.value = false;
      modalTask.value = { id: null, title: '', body: '' };
      isModalOpen.value = true;
    };

    const openEditModal = (task) => {
      isEditing.value = true;
      modalTask.value = { ...task };
      isModalOpen.value = true;
    };

    const closeModal = () => {
      isModalOpen.value = false;
      modalTask.value = { id: null, title: '', body: '' };
      errors.value = { title: '', body: '' };
    };

    const createTask = () => {
      if (!validateForm()) return;

      const newTask = {
        id: Date.now(), // Unique ID
        title: modalTask.value.title,
        body: modalTask.value.body,
      };

      tasks.value.unshift(newTask);
      saveTasks();
      closeModal();
    };

    const updateTask = () => {
      if (!validateForm()) return;

      const index = tasks.value.findIndex((task) => task.id === modalTask.value.id);
      if (index !== -1) {
        tasks.value[index] = { ...modalTask.value };
        saveTasks();
        closeModal();
      }
    };

    const deleteTask = (id) => {
      tasks.value = tasks.value.filter((task) => task.id !== id);
      saveTasks();
    };

    return {
      tasks,
      isModalOpen,
      isEditing,
      modalTask,
      errors,
      openCreateModal,
      openEditModal,
      closeModal,
      createTask,
      updateTask,
      deleteTask,
    };
  },
};
</script>