<template>
  <div class="min-h-screen bg-gray-50 dark:bg-gray-900 text-gray-900 dark:text-gray-100">
    <header class="bg-white dark:bg-gray-800 shadow-sm sticky top-0 z-10">
      <div class="max-w-5xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex justify-between h-16 items-center">
          <router-link to="/" class="text-xl font-bold text-indigo-600 dark:text-indigo-400">
            XCPC岁月史书
          </router-link>
          <nav>
            <router-link to="/" class="text-gray-500 hover:text-gray-700 dark:hover:text-gray-300">首页</router-link>
          </nav>
        </div>
      </div>
    </header>

    <main class="max-w-5xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
      <router-view></router-view>
    </main>

    <footer class="mt-auto py-6 border-t border-gray-200 dark:border-gray-700 text-center text-gray-500 text-sm">
      © 2025-{{ new Date().getFullYear() }} XCPC岁月史书 - Silencer76
    </footer>
  </div>
</template>

<script setup>
</script>
