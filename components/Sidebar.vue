<template>
  <aside 
    :class="[
      'sidebar border-r ring-default transition-all duration-300',
      isCollapsed ? 'w-[70px] min-w-[70px]' : 'w-[250px] min-w-[250px]'
    ]"
  >
    <div class="flex flex-col h-full grow">
      <!-- Logo -->
      <div class="text-center"
      :class="isCollapsed ? 'px-3 py-3' : 'px-4 py-6'" >
        <NuxtImg src="/logo.svg" alt="Logo" class="mx-auto" :class="isCollapsed ? 'max-h-10' : 'max-h-14'" />
      </div>
      
      <!-- Menu -->
      <div class="flex-1 overflow-y-auto">
        <UNavigationMenu 
          orientation="vertical" 
          :class="isCollapsed ? 'px-0 text-center' : 'px-2'"
          :items="navLinks" 
          :collapsed="isCollapsed"           
          :ui="{
            linkLeadingIcon: isCollapsed ? 'text-2xl' : 'text-lg',
            wrapper: isCollapsed ? 'justify-center' : '',
            link: isCollapsed ? 'justify-center' : '',
            
          }"
        />
      </div>
      
      <!-- Bottom Controls -->
      <div class="border-t ring-default py-2">
        <!-- Toggle Theme Button -->
        <div class="mb-2 flex justify-center">
          <UButton 
            :icon="$colorMode.value === 'light' ? 'i-heroicons-moon' : 'i-heroicons-sun'" 
            variant="ghost" 
            color="gray" 
            aria-label="Toggle theme" 
            @click="toggleColorMode"
          />
          <!-- <select class="ring-default text-sm" v-model="$colorMode.preference">
            <option value="system">System</option>
            <option value="light">Light</option>
            <option value="dark">Dark</option>
            <option value="sepia">Sepia</option>
          </select> -->
        </div>
        
        <!-- Toggle Sidebar Button -->
        <UButton 
          block
          variant="soft" 
          color="gray" 
          @click="toggleSidebar"
          :icon="isCollapsed ? 'i-heroicons-arrow-right' : 'i-heroicons-arrow-left'"
          size="sm"
        >
          <span v-if="!isCollapsed">Collapse</span>
        </UButton>
      </div>
    </div>
  </aside>
</template>

<script setup lang="ts">
const config = useRuntimeConfig();
const wpApiUrl = config.public.wordpressApiUrl;

const isCollapsed = ref(false);
const menu = ref<any[]>([]);
const error = ref<string | null>(null);
const $colorMode = useColorMode();

// Navigation links computed from WP menu items
const navLinks = computed(() => {
  if (error.value) {
    return [{ label: 'Error loading menu', icon: 'i-heroicons-exclamation-triangle', to: '#' }];
  }
  
  if (menu.value.length === 0) {
    return [{ label: 'Loading...', icon: 'i-heroicons-arrow-path', to: '#' }];
  }
  
  return menu.value.map(item => {
    console.log(isCollapsed.value);
    return {
      label: item.post_title,
      icon: item.icon,
      to: `${item.url}`
    }
  });
});


// Function to toggle sidebar collapse state
const toggleSidebar = () => {
  isCollapsed.value = !isCollapsed.value;
};

// Function to toggle theme
function toggleColorMode() {
  $colorMode.preference = $colorMode.value === 'dark' ? 'light' : 'dark';
}

// Fetch menu items from WordPress
onMounted(async () => {
  try {
    // Using standard WP REST API to get pages as menu items
    const response = await fetch(`${wpApiUrl}/menus/87`);
    if (!response.ok) {
      throw new Error('Failed to fetch menu data');
    }
    const data = await response.json();
    menu.value = data || [];
  } catch (err) {
    console.error('Error fetching menu:', err);
    error.value = 'Failed to load menu items';
  }
});

// Expose the collapse state for parent components
defineExpose({
  isCollapsed
});
</script>

<style scoped>
.sidebar {
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
}
</style> 