<template>
  <div>
    <UCard class="mb-6">
      <template #header>
        <div class="flex items-center justify-between">
          <h1 class="text-xl font-bold">Welcome to the Homepage</h1>
        </div>
      </template>
      <p>This page demonstrates the sidebar with WordPress menu integration.</p>
      <template #footer>
        <div class="flex items-center gap-2">
          <UButton color="primary" to="https://example.com">
            Visit WordPress
          </UButton>
          <UButton color="primary" to="/cv">
            Тестовое резюме
          </UButton>
        </div>
      </template>
    </UCard>
    
    <!-- Тестовый компонент для проверки переменных окружения -->
    <EnvTest class="mb-6" />
    
    <div class="flex items-center my-6">
      <div class="h-px bg-gray-200 dark:bg-gray-700 flex-grow"></div>
      <span class="px-3 text-sm text-gray-500 dark:text-gray-400">Demo Content</span>
      <div class="h-px bg-gray-200 dark:bg-gray-700 flex-grow"></div>
    </div>
    
    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mt-6">
      <UCard>
        <template #header>
          <h3 class="text-lg font-semibold">Sidebar Features</h3>
        </template>
        <ul class="list-disc pl-5 space-y-2">
          <li>Responsive design</li>
          <li>Collapsible sidebar</li>
          <li>WordPress API integration</li>
          <li>Dark/Light mode toggle</li>
        </ul>
      </UCard>
      
      <UCard>
        <template #header>
          <h3 class="text-lg font-semibold">Nuxt UI Components</h3>
        </template>
        <ul class="list-disc pl-5 space-y-2">
          <li>Custom sidebar with responsive design</li>
          <li>UCards for content display</li>
          <li>UButtons for actions</li>
          <li>UIcon for menu icons</li>
        </ul>
      </UCard>
    </div>
  </div>
</template>

<script setup lang="ts">
definePageMeta({
  layout: 'default'
});
</script> 