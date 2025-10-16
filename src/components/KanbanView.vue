<script setup>
import { ref } from 'vue'

const props = defineProps({
  tasks: Array
})

const emit = defineEmits(['add-task', 'update-task'])

const showModal = ref(false)
const draggedTask = ref(null)
const newTask = ref({
  task: '',
  developers: '',
  status: 'Ready to start',
  priority: 'Medium',
  type: 'Feature Enhancements',
  date: new Date().toISOString().split('T')[0],
  estimatedSP: 0,
  actualSP: 0
})

const statusColumns = [
  'Ready to start',
  'In Progress',
  'Waiting for review',
  'Pending Deploy',
  'Done',
  'Stuck'
]

const priorityOptions = ['Critical', 'High', 'Medium', 'Low', 'Best Effort']
const typeOptions = ['Feature Enhancements', 'Other', 'Bug']

const priorityColors = {
  'Critical': '#dc2626',
  'High': '#f97316',
  'Medium': '#eab308',
  'Low': '#06b6d4',
  'Best Effort': '#6b7280'
}

const typeColors = {
  'Feature Enhancements': '#8b5cf6',
  'Other': '#64748b',
  'Bug': '#ef4444'
}

const statusBgColors = {
  'Ready to start': '#3b82f6',
  'In Progress': '#f97316',
  'Waiting for review': '#22d3ee',
  'Pending Deploy': '#8b5cf6',
  'Done': '#10b981',
  'Stuck': '#ef4444'
}

function getTasksByStatus(status) {
  let filtered = []
  for(let i = 0; i < props.tasks.length; i++) {
    if(props.tasks[i].status === status) {
      filtered.push(props.tasks[i])
    }
  }
  return filtered
}

function openModal() {
  showModal.value = true
}

function closeModal() {
  showModal.value = false
  resetForm()
}

function resetForm() {
  newTask.value = {
    task: '',
    developers: '',
    status: 'Ready to start',
    priority: 'Medium',
    type: 'Feature Enhancements',
    date: new Date().toISOString().split('T')[0],
    estimatedSP: 0,
    actualSP: 0
  }
}

function handleSubmit() {
  if (!newTask.value.task) {
    alert('Task name is required!')
    return
  }

  let taskData = { ...newTask.value }
  taskData.developers = newTask.value.developers.split(',').map(d => d.trim()).filter(d => d)

  emit('add-task', taskData)
  closeModal()
}

function handleDragStart(task) {
  draggedTask.value = task
}

function handleDragOver(e) {
  e.preventDefault()
}

function handleDrop(e, newStatus) {
  e.preventDefault()
  
  if (draggedTask.value && draggedTask.value.status !== newStatus) {
    let updatedTask = { ...draggedTask.value }
    updatedTask.status = newStatus
    emit('update-task', updatedTask)
  }
  
  draggedTask.value = null
}

function handleDragEnd() {
  draggedTask.value = null
}

function getStatusEmoji(status) {
  if(status === 'Ready to start') return '🚀'
  if(status === 'In Progress') return '⚡'
  if(status === 'Waiting for review') return '👀'
  if(status === 'Pending Deploy') return '🚀'
  if(status === 'Done') return '✅'
  if(status === 'Stuck') return '🚨'
  return '📋'
}
</script>

<template>
  <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
    <div 
      v-for="status in statusColumns" 
      :key="status"
      class="rounded-lg overflow-hidden"
      @dragover="handleDragOver"
      @drop="handleDrop($event, status)"
    >
      <div class="p-3 text-white font-semibold text-sm flex items-center justify-between"
           :style="{ backgroundColor: statusBgColors[status] }">
        <span>{{ status }} {{ getTasksByStatus(status).length }}</span>
      </div>

      <div class="p-3 space-y-3 min-h-[300px]" style="background-color: #2a2d3e;">
        <div 
          v-for="task in getTasksByStatus(status)" 
          :key="task.id"
          class="rounded-lg p-3 cursor-move transition-all"
          style="background-color: #1e2139;"
          draggable="true"
          @dragstart="handleDragStart(task)"
          @dragend="handleDragEnd"
        >
          <h4 class="text-gray-300 mb-3 text-sm font-medium">{{ task.task }}</h4>

          <div class="flex flex-wrap gap-2 mb-3">
            <span 
              class="inline-block px-2 py-1 rounded text-white text-xs"
              :style="{ backgroundColor: priorityColors[task.priority] }"
            >
              {{ task.priority }}
            </span>
            
            <span 
              class="inline-block px-2 py-1 rounded text-white text-xs"
              :style="{ backgroundColor: typeColors[task.type] }"
            >
              {{ task.type }}
            </span>
          </div>

          <div class="text-xs text-gray-400 mb-3">
            {{ task.estimatedSP }} SP
          </div>

          <div class="flex items-center gap-2">
            <div class="w-6 h-6 rounded-full bg-gray-600 flex items-center justify-center text-xs text-white">
              {{ task.developers[0] ? task.developers[0].charAt(0) : 'U' }}
            </div>
            <button class="p-1 text-gray-500 hover:text-gray-300">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 5v.01M12 12v.01M12 19v.01M12 6a1 1 0 110-2 1 1 0 010 2zm0 7a1 1 0 110-2 1 1 0 010 2zm0 7a1 1 0 110-2 1 1 0 010 2z"/>
              </svg>
            </button>
            <button class="p-1 text-gray-500 hover:text-gray-300 ml-auto">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"/>
              </svg>
            </button>
          </div>
        </div>

        <div 
          v-if="getTasksByStatus(status).length === 0"
          class="text-center text-gray-500 text-sm py-12"
        >
          Drop tasks here
        </div>
      </div>
    </div>
  </div>

    <!-- Modal -->
    <div 
      v-if="showModal"
      class="fixed inset-0 bg-black bg-opacity-60 flex items-center justify-center z-50 p-4 backdrop-blur-sm"
      @click.self="closeModal"
    >
      <div class="bg-white rounded-2xl shadow-2xl w-full max-w-lg mx-4 max-h-[90vh] overflow-y-auto">
        <div class="p-6 sm:p-8">
          <div class="flex items-center justify-between mb-6">
            <h2 class="text-2xl sm:text-3xl font-bold bg-gradient-to-r from-blue-600 to-purple-600 bg-clip-text text-transparent">Add New Task</h2>
            <button @click="closeModal" class="text-gray-400 hover:text-gray-600 transition-colors">
              <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
              </svg>
            </button>
          </div>

          <form @submit.prevent="handleSubmit" class="space-y-4">
            <div>
              <label class="block text-sm font-bold text-gray-700 mb-2">📝 Task Name *</label>
              <input 
                v-model="newTask.task"
                type="text"
                required
                class="w-full px-4 py-3 border-2 border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-all text-sm"
                placeholder="What needs to be done?"
              />
            </div>

            <div>
              <label class="block text-sm font-bold text-gray-700 mb-2">👥 Developers</label>
              <input 
                v-model="newTask.developers"
                type="text"
                class="w-full px-4 py-3 border-2 border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-all text-sm"
                placeholder="John, Jane, Bob..."
              />
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
              <div>
                <label class="block text-sm font-bold text-gray-700 mb-2">📊 Status</label>
                <select 
                  v-model="newTask.status"
                  class="w-full px-4 py-3 border-2 border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all text-sm"
                >
                  <option v-for="status in statusColumns" :key="status" :value="status">
                    {{ status }}
                  </option>
                </select>
              </div>

              <div>
                <label class="block text-sm font-bold text-gray-700 mb-2">⚡ Priority</label>
                <select 
                  v-model="newTask.priority"
                  class="w-full px-4 py-3 border-2 border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all text-sm"
                >
                  <option v-for="priority in priorityOptions" :key="priority" :value="priority">
                    {{ priority }}
                  </option>
                </select>
              </div>
            </div>

            <div>
              <label class="block text-sm font-bold text-gray-700 mb-2">🏷️ Type</label>
              <select 
                v-model="newTask.type"
                class="w-full px-4 py-3 border-2 border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all text-sm"
              >
                <option v-for="type in typeOptions" :key="type" :value="type">
                  {{ type }}
                </option>
              </select>
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
              <div>
                <label class="block text-sm font-bold text-gray-700 mb-2">📅 Date</label>
                <input 
                  v-model="newTask.date"
                  type="date"
                  class="w-full px-4 py-3 border-2 border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all text-sm"
                />
              </div>

              <div>
                <label class="block text-sm font-bold text-gray-700 mb-2">📈 Est. SP</label>
                <input 
                  v-model.number="newTask.estimatedSP"
                  type="number"
                  min="0"
                  class="w-full px-4 py-3 border-2 border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all text-sm"
                />
              </div>

              <div>
                <label class="block text-sm font-bold text-gray-700 mb-2">✅ Actual SP</label>
                <input 
                  v-model.number="newTask.actualSP"
                  type="number"
                  min="0"
                  class="w-full px-4 py-3 border-2 border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all text-sm"
                />
              </div>
            </div>

            <div class="flex gap-3 pt-6">
              <button 
                type="submit"
                class="flex-1 px-6 py-3 bg-gradient-to-r from-green-600 to-emerald-600 text-white rounded-xl hover:from-green-700 hover:to-emerald-700 transition-all duration-200 shadow-md hover:shadow-lg font-bold"
              >
                ✨ Create Task
              </button>
              <button 
                type="button"
                @click="closeModal"
                class="flex-1 px-6 py-3 bg-gray-200 text-gray-700 rounded-xl hover:bg-gray-300 transition-all duration-200 font-bold"
              >
                Cancel
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
</template>
