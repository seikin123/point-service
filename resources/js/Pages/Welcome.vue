<script setup lang="ts">
import { ref } from 'vue';
import { Head } from '@inertiajs/vue3';

interface Props {
  message?: string;
}

const props = withDefaults(defineProps<Props>(), {
  message: 'Welcome to Laravel + Vue3 + TypeScript + Tailwind!'
});

const count = ref<number>(0);

const increment = (): void => {
  count.value++;
};
</script>

<template>
  <Head title="Welcome" />

  <div class="min-h-screen bg-gradient-to-br from-blue-500 to-purple-600 flex items-center justify-center p-4">
    <div class="bg-white rounded-lg shadow-2xl p-8 max-w-md w-full">
      <h1 class="text-3xl font-bold text-gray-800 mb-4">
        {{ message }}
      </h1>

      <div class="mb-6">
        <p class="text-gray-600 mb-2">Counter: <span class="font-bold text-2xl text-blue-600">{{ count }}</span></p>
        <button
          @click="increment"
          class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded transition duration-200"
        >
          Increment
        </button>
      </div>

      <div class="border-t pt-4">
        <p class="text-sm text-gray-500">
          ✅ Vue 3 + Composition API<br>
          ✅ TypeScript<br>
          ✅ Tailwind CSS<br>
          ✅ Inertia.js
        </p>
      </div>
    </div>
  </div>
</template>