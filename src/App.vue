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
const kanbanView = ref(null)

onMounted(() => {
  fetch('https://mocki.io/v1/f7861fc0-9071-4034-afed-777f3b590c3c')
    .then((response) => {
      return response.json()
    })
    .then((result) => {
      if (result && result.response && Array.isArray(result.data)) {
        let tmp = []
        for (let i = 0; i < result.data.length; i++) {
          const item = result.data[i]
          const devsStr = item.developer
          let devsArr = []
          if (devsStr && typeof devsStr === 'string') {
            devsArr = devsStr.split(',').map(function(d){ return d.trim() })
          }
          const one = {
            id: i + 1,
            task: item.title,
            developers: devsArr,
            status: item.status,
            priority: item.priority,
            type: item.type,
            date: '',
            estimatedSP: item['Estimated SP'],
            actualSP: item['Actual SP'],
            comments: []
          }
          tmp.push(one)
        }
        tasks.value = tmp
      }
    })
    .catch(() => {
      // do nothing for now
    })
})

const allDevelopers = computed(() => {
  const result = []
  const list = tasks.value || []
  for (let i = 0; i < list.length; i++) {
    const t = list[i]
    if (t && Array.isArray(t.developers)) {
      for (let j = 0; j < t.developers.length; j++) {
        const name = t.developers[j]
        if (result.indexOf(name) === -1) {
          result.push(name)
        }
      }
    }
  }
  return result
})

const filteredTasks = computed(() => {
  let result = []
  const src = tasks.value || []
  for (let i = 0; i < src.length; i++) {
    result.push(src[i])
  }

  if (searchQuery.value) {
    const q = (searchQuery.value + '').toLowerCase()
    const temp = []
    for (let k = 0; k < result.length; k++) {
      const t = result[k]
      const name = (t && t.task ? (t.task + '').toLowerCase() : '')
      if (name.indexOf(q) !== -1) {
        temp.push(t)
      }
    }
    result = temp
  }

  if ((selectedDevelopers.value || []).length > 0) {
    const wanted = selectedDevelopers.value
    const temp2 = []
    for (let m = 0; m < result.length; m++) {
      const t2 = result[m]
      if (!Array.isArray(t2.developers)) continue
      let ok = false
      for (let n = 0; n < t2.developers.length; n++) {
        const dev = t2.developers[n]
        if (wanted.indexOf(dev) !== -1) { ok = true; break }
      }
      if (ok) temp2.push(t2)
    }
    result = temp2
  }

  if ((sortConfig.value || []).length > 0) {
    // manual selection-sort like approach
    for (let i = 0; i < result.length; i++) {
      let idxMin = i
      for (let j = i + 1; j < result.length; j++) {
        let cmp = 0
        for (let k = 0; k < sortConfig.value.length; k++) {
          const config = sortConfig.value[k]
          const a = result[idxMin]
          const b = result[j]
          let valA = a[config.field]
          let valB = b[config.field]
          if (valA === valB) { continue }
          if (config.direction === 'asc') {
            cmp = (valA > valB) ? 1 : -1
          } else {
            cmp = (valA < valB) ? 1 : -1
          }
          break
        }
        if (cmp > 0) {
          idxMin = j
        }
      }
      if (idxMin !== i) {
        const tmp = result[i]
        result[i] = result[idxMin]
        result[idxMin] = tmp
      }
    }
  }

  return result
})

function addNewTask(task) {
  const newObj = {
    id: Date.now(),
    task: task ? task.task : '',
    developers: task && task.developers ? task.developers : [],
    status: task ? task.status : '',
    priority: task ? task.priority : '',
    type: task ? task.type : '',
    date: task ? task.date : '',
    estimatedSP: task ? task.estimatedSP : 0,
    actualSP: task ? task.actualSP : 0,
    comments: task && task.comments ? task.comments : []
  }
  tasks.value.unshift(newObj)
}

function updateTask(t) {
  const idx = tasks.value.findIndex(x => x.id === t.id)
  if (idx !== -1) {
    const cur = tasks.value[idx]
    cur.task = t.task
    cur.developers = t.developers
    cur.status = t.status
    cur.priority = t.priority
    cur.type = t.type
    cur.date = t.date
    cur.estimatedSP = t.estimatedSP
    cur.actualSP = t.actualSP
    cur.comments = t.comments
  }
}

function deleteTask(id) {
  const idx = tasks.value.findIndex(t => t.id === id)
  if (idx !== -1) {
    tasks.value.splice(idx, 1)
  }
}

function changeView(v) {
  if (v === 'table') {
    currentView.value = 'table'
  } else if (v === 'kanban') {
    currentView.value = 'kanban'
  } else {
    currentView.value = v
  }
}

function toggleSearch() {
  if (showSearchModal.value) {
    showSearchModal.value = false
  } else {
    showSearchModal.value = true
  }
}

function togglePerson() {
  if (showPersonModal.value) {
    showPersonModal.value = false
  } else {
    showPersonModal.value = true
  }
}

function toggleSort() {
  if (showSortModal.value) {
    showSortModal.value = false
  } else {
    showSortModal.value = true
  }
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

function isDevSelected(dev) {
  const arr = selectedDevelopers.value || []
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === dev) return true
  }
  return false
}

function addSortColumn(field) {
  let idx = -1
  for (let i = 0; i < sortConfig.value.length; i++) {
    if (sortConfig.value[i].field === field) { idx = i; break }
  }
  if (idx !== -1) {
    if (sortConfig.value[idx].direction === 'asc') {
      sortConfig.value[idx].direction = 'desc'
    } else {
      const temp = []
      for (let j = 0; j < sortConfig.value.length; j++) {
        if (j !== idx) temp.push(sortConfig.value[j])
      }
      sortConfig.value = temp
    }
  } else {
    const added = { field: field, direction: 'asc' }
    const next = []
    for (let k = 0; k < sortConfig.value.length; k++) next.push(sortConfig.value[k])
    next.push(added)
    sortConfig.value = next
  }
}

function getSortIndex(field) {
  for (let i = 0; i < sortConfig.value.length; i++) {
    if (sortConfig.value[i].field === field) return i
  }
  return -1
}

function getSortDirection(field) {
  let dir = ''
  for (let i = 0; i < sortConfig.value.length; i++) {
    if (sortConfig.value[i].field === field) {
      dir = sortConfig.value[i].direction
      break
    }
  }
  return dir
}

function handleNewTask() {
  if (currentView.value === 'table' && tableView.value) {
    tableView.value.toggleNewTask()
  } else if (currentView.value === 'kanban' && kanbanView.value) {
    kanbanView.value.openModal()
  }
}
</script>

<template>
  <div class="min-h-screen" style="background-color: #1a1d2e;">
    <div class="container mx-auto px-3 sm:px-4 lg:px-6 py-4 sm:py-6 max-w-full">
      
      <div class="mb-4">
        <div class="flex items-center justify-between mb-4 px-2">
          <div class="flex items-center gap-4">
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
          </button>

          <button 
            @click="toggleSearch"
            class="px-4 py-2 rounded-md font-medium text-sm transition-all flex items-center gap-2 text-gray-300 hover:text-white"
            style="background-color: #2a2d3e;"
          >
            <span>Search</span>
          </button>

          <button 
            @click="togglePerson"
            class="px-4 py-2 rounded-md font-medium text-sm transition-all flex items-center gap-2 text-gray-300 hover:text-white"
            style="background-color: #2a2d3e;"
          >
            <span>Person</span>
          </button>

          <button 
            @click="toggleSort"
            class="px-4 py-2 rounded-md font-medium text-sm transition-all flex items-center gap-2 text-gray-300 hover:text-white"
            style="background-color: #2a2d3e;"
          >
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
        ref="kanbanView"
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
              :checked="isDevSelected(dev)"
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
               :class="getSortIndex(field) !== -1 ? 'bg-gray-700' : 'bg-gray-800'">
            <span>{{ field }}</span>
            <span v-if="getSortIndex(field) !== -1">
              {{ getSortDirection(field) === 'asc' ? '↑' : '↓' }}
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
