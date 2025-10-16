<script setup>
import { ref, computed, onMounted } from 'vue'
import TableView from './components/TableView.vue'
import KanbanView from './components/KanbanView.vue'

const currentView = ref('table')
const tasks = ref([])
const searchQuery = ref('')
const selectedDevelopers = ref([])
const sortConfig = ref([])
const showSearchModal = ref(false)
const showPersonModal = ref(false)
const showSortModal = ref(false)
const tableView = ref(null)

onMounted(async () => {
  try {
    const response = await fetch('https://mocki.io/v1/4e602db4-efab-438f-a664-bec50fc16f7e')
    const data = await response.json()
    tasks.value = data
  } catch (error) {
    tasks.value = [
      {
        id: 1,
        task: 'New Task',
        developers: [],
        status: 'In Progress',
        priority: 'Medium',
        type: 'Feature Enhancements',
        date: '2024-10-20',
        estimatedSP: 0,
        actualSP: 0
      },
      {
        id: 2,
        task: 'New task',
        developers: [],
        status: 'Waiting for review',
        priority: 'Medium',
        type: 'Feature Enhancements',
        date: '2024-10-18',
        estimatedSP: 0,
        actualSP: 0
      },
      {
        id: 3,
        task: 'New task',
        developers: [],
        status: 'Ready to start',
        priority: 'Best Effort',
        type: 'Feature Enhancements',
        date: '2024-10-15',
        estimatedSP: 0,
        actualSP: 0
      },
      {
        id: 4,
        task: 'Committed Feature',
        developers: [],
        status: 'Ready to start',
        priority: 'High',
        type: 'Other',
        date: '2024-10-22',
        estimatedSP: 2,
        actualSP: 1.5
      }
    ]
  }
})

const allDevelopers = computed(() => {
  const devs = new Set()
  tasks.value.forEach(t => {
    if (Array.isArray(t.developers)) {
      t.developers.forEach(d => devs.add(d))
    }
  })
  return Array.from(devs)
})

const filteredTasks = computed(() => {
  let result = [...tasks.value]
  
  if (searchQuery.value) {
    const q = searchQuery.value.toLowerCase()
    result = result.filter(t => t.task.toLowerCase().includes(q))
  }
  
  if (selectedDevelopers.value.length > 0) {
    result = result.filter(t => {
      if (!Array.isArray(t.developers)) return false
      return t.developers.some(d => selectedDevelopers.value.includes(d))
    })
  }
  
  if (sortConfig.value.length > 0) {
    result.sort((a, b) => {
      for (let i = 0; i < sortConfig.value.length; i++) {
        const config = sortConfig.value[i]
        let valA = a[config.field]
        let valB = b[config.field]
        
        if (valA === valB) continue
        
        if (config.direction === 'asc') {
          return valA > valB ? 1 : -1
        } else {
          return valA < valB ? 1 : -1
        }
      }
      return 0
    })
  }
  
  return result
})

function addNewTask(task) {
  tasks.value.unshift({ id: Date.now(), ...task })
}

function updateTask(t) {
  const idx = tasks.value.findIndex(x => x.id === t.id)
  if (idx !== -1) {
    tasks.value[idx] = t
  }
}

function deleteTask(id) {
  const idx = tasks.value.findIndex(t => t.id === id)
  if (idx !== -1) {
    tasks.value.splice(idx, 1)
  }
}

function changeView(v) {
  currentView.value = v
}

function toggleSearch() {
  showSearchModal.value = !showSearchModal.value
}

function togglePerson() {
  showPersonModal.value = !showPersonModal.value
}

function toggleSort() {
  showSortModal.value = !showSortModal.value
}

function clearFilters() {
  searchQuery.value = ''
  selectedDevelopers.value = []
}

function toggleDeveloper(dev) {
  const idx = selectedDevelopers.value.indexOf(dev)
  if (idx > -1) {
    selectedDevelopers.value.splice(idx, 1)
  } else {
    selectedDevelopers.value.push(dev)
  }
}

function addSortColumn(field) {
  const existing = sortConfig.value.find(s => s.field === field)
  if (existing) {
    if (existing.direction === 'asc') {
      existing.direction = 'desc'
    } else {
      sortConfig.value = sortConfig.value.filter(s => s.field !== field)
    }
  } else {
    sortConfig.value.push({ field, direction: 'asc' })
  }
}

function handleNewTask() {
  if (tableView.value) {
    tableView.value.toggleNewTask()
  }
}
</script>

<template>
  <div class="min-h-screen" style="background-color: #1a1d2e;">
    <div class="container mx-auto px-3 sm:px-4 lg:px-6 py-4 sm:py-6 max-w-full">
      
      <div class="mb-4">
        <div class="flex items-center justify-between mb-4 px-2">
          <div class="flex items-center gap-4">
            <span class="text-gray-400 text-sm">🏠</span>
            <button 
              @click="changeView('table')"
              class="px-4 py-2 rounded-md font-medium text-sm transition-all"
              :class="currentView === 'table' 
                ? 'text-white border-b-2 border-blue-500' 
                : 'text-gray-400 hover:text-gray-300'"
            >
              Main Table
            </button>
            <button 
              @click="changeView('kanban')"
              class="px-4 py-2 rounded-md font-medium text-sm transition-all"
              :class="currentView === 'kanban' 
                ? 'text-white border-b-2 border-blue-500' 
                : 'text-gray-400 hover:text-gray-300'"
            >
              Kanban
            </button>
          </div>
        </div>

        <div class="px-2 py-4 flex flex-wrap items-center gap-3">
          <button 
            @click="handleNewTask"
            class="px-4 py-2 rounded-md font-medium text-sm transition-all flex items-center gap-2 text-white hover:opacity-90"
            style="background-color: #2563eb;"
          >
            <span>New task</span>
            <span class="text-xs">▼</span>
          </button>

          <button 
            @click="toggleSearch"
            class="px-4 py-2 rounded-md font-medium text-sm transition-all flex items-center gap-2 text-gray-300 hover:text-white"
            style="background-color: #2a2d3e;"
          >
            <span>🔍</span>
            <span>Search</span>
          </button>

          <button 
            @click="togglePerson"
            class="px-4 py-2 rounded-md font-medium text-sm transition-all flex items-center gap-2 text-gray-300 hover:text-white"
            style="background-color: #2a2d3e;"
          >
            <span>👤</span>
            <span>Person</span>
          </button>

          <button 
            @click="toggleSort"
            class="px-4 py-2 rounded-md font-medium text-sm transition-all flex items-center gap-2 text-gray-300 hover:text-white"
            style="background-color: #2a2d3e;"
          >
            <span>⇅</span>
            <span>Sort</span>
          </button>
        </div>
      </div>

      <TableView 
        ref="tableView"
        v-if="currentView === 'table'"
        :tasks="filteredTasks"
        :sortConfig="sortConfig"
        @add-task="addNewTask"
        @update-task="updateTask"
        @delete-task="deleteTask"
        @update-sort="sortConfig = $event"
      />
      
      <KanbanView 
        v-else
        :tasks="filteredTasks"
        @add-task="addNewTask"
        @update-task="updateTask"
      />
    </div>

    <div v-if="showSearchModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50" @click.self="showSearchModal = false">
      <div class="rounded-lg p-6 w-full max-w-md mx-4" style="background-color: #2a2d3e;">
        <h3 class="text-white font-bold mb-4">Search Tasks</h3>
        <input 
          v-model="searchQuery"
          type="text"
          placeholder="Type to search..."
          class="w-full px-4 py-2 rounded bg-gray-800 text-white border border-gray-600 focus:outline-none focus:border-blue-500"
        />
        <div class="flex gap-2 mt-4">
          <button @click="showSearchModal = false" class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700">
            Apply
          </button>
          <button @click="clearFilters(); showSearchModal = false" class="px-4 py-2 bg-gray-600 text-white rounded hover:bg-gray-700">
            Clear
          </button>
        </div>
      </div>
    </div>

    <div v-if="showPersonModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50" @click.self="showPersonModal = false">
      <div class="rounded-lg p-6 w-full max-w-md mx-4" style="background-color: #2a2d3e;">
        <h3 class="text-white font-bold mb-4">Filter by Developer</h3>
        <div class="space-y-2 max-h-64 overflow-y-auto">
          <div v-for="dev in allDevelopers" :key="dev" class="flex items-center gap-2">
            <input 
              type="checkbox"
              :checked="selectedDevelopers.includes(dev)"
              @change="toggleDeveloper(dev)"
              class="w-4 h-4"
            />
            <label class="text-gray-300">{{ dev }}</label>
          </div>
        </div>
        <div class="flex gap-2 mt-4">
          <button @click="showPersonModal = false" class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700">
            Apply
          </button>
          <button @click="selectedDevelopers = []; showPersonModal = false" class="px-4 py-2 bg-gray-600 text-white rounded hover:bg-gray-700">
            Clear
          </button>
        </div>
      </div>
    </div>

    <div v-if="showSortModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50" @click.self="showSortModal = false">
      <div class="rounded-lg p-6 w-full max-w-md mx-4" style="background-color: #2a2d3e;">
        <h3 class="text-white font-bold mb-4">Sort Columns</h3>
        <div class="space-y-2">
          <div v-for="field in ['task', 'developers', 'status', 'priority', 'type', 'date', 'estimatedSP', 'actualSP']" 
               :key="field"
               @click="addSortColumn(field)"
               class="px-4 py-2 rounded cursor-pointer text-gray-300 hover:bg-gray-700 flex items-center justify-between"
               :class="sortConfig.find(s => s.field === field) ? 'bg-gray-700' : 'bg-gray-800'">
            <span>{{ field }}</span>
            <span v-if="sortConfig.find(s => s.field === field)">
              {{ sortConfig.find(s => s.field === field).direction === 'asc' ? '↑' : '↓' }}
            </span>
          </div>
        </div>
        <div class="flex gap-2 mt-4">
          <button @click="showSortModal = false" class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700">
            Apply
          </button>
          <button @click="sortConfig = []; showSortModal = false" class="px-4 py-2 bg-gray-600 text-white rounded hover:bg-gray-700">
            Clear
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
select[multiple] {
  background-image: none;
}

@media (max-width: 640px) {
  select[multiple] {
    height: 42px !important;
  }
}
</style>
