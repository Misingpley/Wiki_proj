<script setup lang="ts">
import { RouterView } from 'vue-router'
</script>

<template>
  <div class="flex h-screen bg-[#0b0f19] text-gray-300 font-sans">
    
    <aside class="w-64 bg-[#111522] border-r border-gray-800 flex flex-col">
      <div class="h-16 flex items-center px-6 border-b border-gray-800">
        <span class="text-white font-bold text-lg flex items-center gap-3">
          <div class="w-7 h-7 bg-indigo-600 rounded flex items-center justify-center text-sm">W</div>
          WikiBase
        </span>
      </div>
      
      <nav class="p-4 flex-1 overflow-y-auto">
        <div class="text-xs text-gray-500 mb-3 uppercase tracking-wider font-semibold">Навігація</div>
        <ul class="space-y-1">
          <li>
            <a href="#" class="flex items-center gap-2 px-3 py-2 text-indigo-400 bg-[#1a1f35] rounded-md text-sm">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"></path></svg>
              Інструкції
            </a>
          </li>
        </ul>
      </nav>
    </aside>

    <div class="flex-1 flex flex-col overflow-hidden">
      
      <header class="h-16 flex items-center justify-between px-6 border-b border-gray-800 bg-[#111522]">
        <div class="w-[400px]">
          <div class="relative">
            <svg class="w-4 h-4 absolute left-3 top-2.5 text-gray-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path></svg>
            <input type="text" placeholder="Пошук по мануалах та IP-адресах (Ctrl+K)..." class="w-full bg-[#0b0f19] border border-gray-700 rounded-md pl-10 pr-4 py-1.5 text-sm text-gray-200 focus:outline-none focus:border-indigo-500">
          </div>
        </div>
        
        <div class="flex items-center gap-3 cursor-pointer hover:opacity-80 transition-opacity">
          <svg class="w-5 h-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"></path></svg>
          <div class="w-8 h-8 rounded-full bg-indigo-500 flex items-center justify-center text-white text-sm font-medium">ОК</div>
          <span class="text-sm font-medium">Олексій</span>
        </div>
      </header>

      <main class="flex-1 overflow-auto p-8 bg-[#0b0f19]">
        <RouterView />
      </main>
      
    </div>
  </div>
</template>

<style>
/* Здесь пока пусто, Tailwind всё делает за нас */
</style>