<script setup lang="ts">
import RichEditor from '@/components/RichEditor.vue'
</script>

<template>
  <div class="h-full flex flex-col max-w-5xl mx-auto w-full">
    <div class="mb-6 flex justify-between items-center">
      <div>
        <h1 class="text-2xl font-bold text-white">Створення статті</h1>
        <p class="text-gray-400 text-sm">Напишіть мануал за допомогою редактора</p>
      </div>
      <button class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-lg font-medium transition-colors">
        Зберегти статтю
      </button>
    </div>
    
    <div class="flex-1 min-h-[600px]">
      <RichEditor />
    </div>
  </div>
</template>