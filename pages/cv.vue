<template>
  <div>
    <div class="container mx-auto py-8 px-4">
      <UCard v-if="loading" class="p-4">
        <USkeleton class="h-32 w-32 rounded-full mx-auto mb-4" />
        <USkeleton class="h-8 w-3/4 mx-auto mb-2" />
        <USkeleton class="h-6 w-1/2 mx-auto mb-4" />
        <USkeleton v-for="i in 5" :key="i" class="h-4 w-full mb-2" />
      </UCard>

      <div v-else-if="error" class="text-center">
        <UAlert type="danger" title="Error" :description="error" />
      </div>

      <template v-else-if="cvData">
        <UCard class="mb-6">
          <div class="flex flex-col md:flex-row items-center md:items-start gap-6 p-4">
            <div class="flex-shrink-0">
              <UAvatar 
                :src="cvData.photo || 'https://ui.nuxt.com/assets/avatar-placeholder.svg'" 
                alt="Profile photo"
                size="2xl"
                class="h-32 w-32"
              />
            </div>
            <div class="flex-grow text-center md:text-left">
              <h1 class="text-2xl font-bold mb-2">{{ cvData.name }}</h1>
              <div class="mb-2">
                <UBadge v-if="cvData.status" color="primary" class="mr-2">{{ cvData.status }}</UBadge>
                <UBadge v-if="cvData.town" color="gray" class="mr-2">{{ cvData.town }}</UBadge>
                <UBadge v-if="cvData.age" color="gray">{{ cvData.age }} лет</UBadge>
              </div>
              <div class="flex flex-col md:flex-row gap-2 md:gap-4 justify-center md:justify-start">
                <div v-if="cvData.phone" class="flex items-center">
                  <UIcon name="i-heroicons-phone" class="mr-1" />
                  <span>{{ cvData.phone }}</span>
                </div>
                <div v-if="cvData.email" class="flex items-center">
                  <UIcon name="i-heroicons-envelope" class="mr-1" />
                  <span>{{ cvData.email }}</span>
                </div>
                <div v-if="cvData.birth_date" class="flex items-center">
                  <UIcon name="i-heroicons-calendar" class="mr-1" />
                  <span>{{ formatDate(cvData.birth_date) }}</span>
                </div>
              </div>
            </div>
            <div class="flex-shrink-0 flex gap-2">
              <UButton color="gray" icon="i-heroicons-printer" variant="ghost" @click="printCV" />
              <UButton color="gray" icon="i-heroicons-document-text" variant="ghost" @click="downloadPDF" />
              <UButton color="gray" icon="i-heroicons-document" variant="ghost" @click="downloadDOC" />
              <UButton color="gray" icon="i-heroicons-pencil-square" variant="ghost" @click="editCV" />
              <UModal v-model:open="isDeleteModalOpen" title="Подтверждение удаления">
                <UButton color="red" icon="i-heroicons-trash" variant="ghost" />
                <template #body>
                  <p class="mb-4">Вы уверены, что хотите удалить этого кандидата?</p>
                  <div class="flex justify-end gap-2">
                    <UButton color="gray" @click="isDeleteModalOpen = false">Отмена</UButton>
                    <UButton color="red" @click="deleteCV">Удалить</UButton>
                  </div>
                </template>
              </UModal>
              
              <UModal v-model:open="isNotesModalOpen" title="Заметки">
                <UButton color="gray" icon="i-heroicons-clipboard-document-list" variant="ghost" />
                <template #body>
                  <UTextarea v-model="notes" placeholder="Введите заметки о кандидате..." rows="5" class="w-full mb-4" />
                  <div class="flex justify-end gap-2">
                    <UButton color="gray" @click="isNotesModalOpen = false">Отмена</UButton>
                    <UButton color="primary" @click="saveNotes">Сохранить</UButton>
                  </div>
                </template>
              </UModal>
              
              <UButton color="primary" icon="i-heroicons-heart" variant="ghost" @click="addToFavorites" />
            </div>
          </div>
        </UCard>

        <UCard class="mb-6">
          <template #header>
            <div class="font-medium">Статус рассмотрения:</div>
          </template>
          <div class="p-4">
            <UProgress :model-value="getStatusProgress()" :indeterminate="false" class="mb-4" />
            <div class="grid grid-cols-1 md:grid-cols-7 gap-2">
              <UBadge :color="getStatusColor('new')" variant="soft" class="text-center">Новое</UBadge>
              <UBadge :color="getStatusColor('viewed')" variant="soft" class="text-center">Просмотрено</UBadge>
              <UBadge :color="getStatusColor('invitation')" variant="soft" class="text-center">Отправлено приглашение</UBadge>
              <UBadge :color="getStatusColor('interview')" variant="soft" class="text-center">Назначено собеседование</UBadge>
              <UBadge :color="getStatusColor('noshow')" variant="soft" class="text-center">Не дошел</UBadge>
              <UBadge :color="getStatusColor('completed')" variant="soft" class="text-center">Проведено собеседование</UBadge>
              <UBadge :color="getStatusColor('rejected')" variant="soft" class="text-center">Отклонено/Отказ</UBadge>
            </div>
          </div>
        </UCard>

        <UCard>
          <template #header>
            <div class="font-medium">Информация о кандидате</div>
          </template>
          <div class="p-4">
            <div v-if="parsedDescription" class="space-y-4">
              <template v-for="(section, index) in parsedDescription" :key="index">
                <div v-if="section.title" class="mb-2">
                  <h3 class="text-lg font-semibold">{{ section.title }}</h3>
                  <div v-if="section.content" class="mt-1">
                    <p v-for="(item, i) in section.content" :key="i" class="mb-1">{{ item }}</p>
                  </div>
                  <div v-if="section.items && section.items.length">
                    <div v-for="(item, i) in section.items" :key="i" class="mb-3 border-b pb-3 last:border-0">
                      <div v-if="item.title" class="font-medium">{{ item.title }}</div>
                      <div v-if="item.subtitle" class="text-sm text-gray-600">{{ item.subtitle }}</div>
                      <div v-if="item.period" class="text-sm text-gray-500">{{ item.period }}</div>
                      <div v-if="item.description" class="mt-1 text-sm">
                        <p v-for="(line, j) in item.description.split('\n')" :key="j">{{ line }}</p>
                      </div>
                    </div>
                  </div>
                </div>
              </template>
            </div>
            <div v-else class="whitespace-pre-line">{{ cvData.description }}</div>
          </div>
        </UCard>

        <UCard v-if="cvData.portfolios" class="mt-6">
          <template #header>
            <div class="font-medium">Файлы портфолио:</div>
          </template>
          <div class="p-4">
            <div v-for="(portfolio, index) in cvData.portfolios" :key="index" class="mb-2">
              <UButton 
                :icon="getFileIcon(portfolio)" 
                variant="ghost" 
                color="primary"
                @click="downloadPortfolio(portfolio)"
              >
                {{ portfolio.name || 'Файл ' + (index + 1) }}
              </UButton>
            </div>
          </div>
        </UCard>
      </template>
    </div>
  </div>
</template>

<script setup lang="ts">
definePageMeta({
  layout: "default",
});

interface CVData {
  id: number;
  name: string;
  description: string;
  date: string;
  status: string;
  portfolios: PortfolioItem[] | null;
  town: string;
  phone: string;
  age: number;
  birth_date: string;
  email: string;
  listing_id: number;
  photo: string;
}

interface PortfolioItem {
  name?: string;
  type?: string;
  url?: string;
  [key: string]: unknown;
}

interface DescriptionSection {
  title?: string;
  content?: string[];
  items?: {
    title?: string;
    subtitle?: string;
    period?: string;
    description?: string;
  }[];
}

const loading = ref(true);
const error = ref('');
const cvData = ref<CVData | null>(null);
const notes = ref('');
const parsedDescription = ref<DescriptionSection[]>([]);
const isDeleteModalOpen = ref(false);
const isNotesModalOpen = ref(false);

onMounted(async () => {
  try {
    const response = await fetch('https://dev.jobcart.ru/wp-json/test/v2/app');
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    cvData.value = await response.json();
    
    if (cvData.value?.description) {
      parseDescription(cvData.value.description);
    }
  } catch (e) {
    error.value = e instanceof Error ? e.message : 'An error occurred while fetching data';
  } finally {
    loading.value = false;
  }
});

const formatDate = (dateString: string) => {
  try {
    const date = new Date(dateString);
    return new Intl.DateTimeFormat('ru-RU', { day: '2-digit', month: '2-digit', year: 'numeric' }).format(date);
  } catch {
    return dateString;
  }
};

const parseDescription = (description: string) => {
  const sections: DescriptionSection[] = [];
  const lines = description.split('\r\n');
  
  const sectionTypes = {
    'Опыт работы:': { title: 'Опыт работы', hasItems: true },
    'Образование:': { title: 'Образование', hasItems: true },
    'Дополнительное образование:': { title: 'Дополнительное образование', hasItems: true },
    'Владение языками:': { title: 'Владение языками', hasItems: false },
    'Ключевые навыки:': { title: 'Ключевые навыки', hasItems: false },
    'Подробнее о навыках:': { title: 'Подробнее о навыках', hasItems: false }
  };
  
  const experienceFields = {
    'Компания:': 'title',
    'Должность:': 'subtitle',
    'Период:': 'period',
    'Описание:': 'description'
  };
  
  let currentSection: DescriptionSection = {};
  let currentExperience: Record<string, string> | null = null;
  
  // Process each line
  for (const line of lines) {
    if (!line) continue;
    
    // Check if line starts a new section
    const sectionMatch = Object.entries(sectionTypes).find(([key]) => line.includes(key));
    if (sectionMatch) {
      const [_, sectionConfig] = sectionMatch;
      currentSection = { 
        title: sectionConfig.title, 
        ...(sectionConfig.hasItems ? { items: [] } : { content: [] })
      };
      sections.push(currentSection);
      continue;
    }
    
    if (currentSection.title === 'Опыт работы') {
      const experienceMatch = Object.entries(experienceFields).find(([key]) => line.startsWith(key));
      if (experienceMatch) {
        const [key, field] = experienceMatch;
        
        if (key === 'Компания:') {
          currentExperience = { title: line.replace(key, '').trim() };
          if (currentSection.items) {
            currentSection.items.push(currentExperience);
          }
        } 
        else if (currentExperience) {
          currentExperience[field] = line.replace(key, '').trim();
        }
        continue;
      }
    }
    
    if (currentSection.content) {
      currentSection.content.push(line);
    } 
    else if (!currentSection.title && line) {
      if (sections.length === 0) {
        currentSection = { title: 'Общая информация', content: [line] };
        sections.push(currentSection);
      }
    }
  }
  
  parsedDescription.value = sections;
};

const getStatusProgress = () => {
  const statusMap: Record<string, number> = {
    'new': 14,
    'viewed': 28,
    'invitation': 42,
    'interview': 56,
    'noshow': 70,
    'completed': 84,
    'rejected': 100
  };
  
  return statusMap[cvData.value?.status || 'new'] || 0;
};

const getStatusColor = (status: string) => {
  if (!cvData.value) return 'gray';
  
  const currentStatus = cvData.value.status;
  const statusOrder = ['new', 'viewed', 'invitation', 'interview', 'noshow', 'completed', 'rejected'];
  
  const currentIndex = statusOrder.indexOf(currentStatus);
  const statusIndex = statusOrder.indexOf(status);
  
  if (statusIndex < currentIndex) return 'green';
  if (statusIndex === currentIndex) return 'blue';
  return 'gray';
};

const getFileIcon = (file: PortfolioItem) => {
  const fileType = file.type || '';
  if (fileType.includes('pdf')) return 'i-heroicons-document-text';
  if (fileType.includes('doc')) return 'i-heroicons-document';
  if (fileType.includes('image')) return 'i-heroicons-photo';
  return 'i-heroicons-paper-clip';
};

const printCV = () => {
  window.print();
};

const downloadPDF = () => {
  alert('PDF download functionality would be implemented here');
};

const downloadDOC = () => {
  alert('DOC download functionality would be implemented here');
};

const editCV = () => {
  alert('Edit functionality would be implemented here');
};

const deleteCV = () => {
  alert('Delete functionality would be implemented here');
};

const addToFavorites = () => {
  alert('Add to favorites functionality would be implemented here');
};

const downloadPortfolio = (portfolio: PortfolioItem) => {
  alert(`Download ${portfolio.name || 'file'} would be implemented here`);
};

const saveNotes = () => {
  alert(`Notes saved: ${notes.value}`);
};
</script>
