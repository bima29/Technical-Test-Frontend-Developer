<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  tasks: Array,
  sortConfig: Array
})

const emit = defineEmits(['add-task', 'update-task', 'delete-task', 'update-sort'])

const editingCell = ref(null)
const editingValue = ref('')
const showNewTaskRow = ref(false)

const newTask = ref({
  task: 'New task',
  developers: [],
  status: 'Ready to start',
  priority: 'Medium',
  type: 'Feature Enhancements',
  date: new Date().toISOString().split('T')[0],
  estimatedSP: 0,
  actualSP: 0
})

const statusOptions = ['Ready to start', 'In Progress', 'Waiting for review', 'Pending Deploy', 'Done', 'Stuck']
const priorityOptions = ['Critical', 'High', 'Medium', 'Low', 'Best Effort']
const typeOptions = ['Feature Enhancements', 'Other', 'Bug']

const statusColors = {
  'Ready to start': '#3b82f6',
  'In Progress': '#06b6d4',
  'Waiting for review': '#f59e0b',
  'Pending Deploy': '#8b5cf6',
  'Done': '#10b981',
  'Stuck': '#ef4444'
}

const priorityColors = {
  'Critical': '#ef4444',
  'High': '#3b82f6',
  'Medium': '#06b6d4',
  'Low': '#10b981',
  'Best Effort': '#ec4899'
}

const typeColors = {
  'Feature Enhancements': '#fb7185',
  'Other': '#a78bfa',
  'Bug': '#ef4444'
}

function handleSort(field) {
  let newConfig = [...props.sortConfig]
  let idx = newConfig.findIndex(c => c.field === field)
  
  if (idx > -1) {
    if (newConfig[idx].direction === 'asc') {
      newConfig[idx].direction = 'desc'
    } else {
      newConfig.splice(idx, 1)
    }
  } else {
    newConfig.push({ field, direction: 'asc' })
  }
  
  emit('update-sort', newConfig)
}

function getSortIndicator(field) {
  const config = props.sortConfig.find(c => c.field === field)
  if (config) {
    return config.direction === 'asc' ? ' ↑' : ' ↓'
  }
  return ''
}

function startEdit(taskId, field, currentValue) {
  editingCell.value = taskId + '-' + field
  editingValue.value = Array.isArray(currentValue) ? currentValue.join(', ') : currentValue
}

function saveEdit(task, field) {
  let val = editingValue.value
  
  if (field === 'developers') {
    val = val.split(',').map(d => d.trim()).filter(d => d)
  } else if (field === 'estimatedSP' || field === 'actualSP') {
    val = parseFloat(val) || 0
  }
  
  emit('update-task', { ...task, [field]: val })
  editingCell.value = null
}

function cancelEdit() {
  editingCell.value = null
}

function toggleNewTask() {
  showNewTaskRow.value = !showNewTaskRow.value
}

function handleAddTask() {
  emit('add-task', { ...newTask.value })
  
  newTask.value = {
    task: 'New task',
    developers: [],
    status: 'Ready to start',
    priority: 'Medium',
    type: 'Feature Enhancements',
    date: new Date().toISOString().split('T')[0],
    estimatedSP: 0,
    actualSP: 0
  }
  showNewTaskRow.value = false
}

function formatDate(dateStr) {
  const d = new Date(dateStr)
  const months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
  return d.getDate() + ' ' + months[d.getMonth()] + ', ' + d.getFullYear()
}

defineExpose({ toggleNewTask })

const statusPercentages = computed(() => {
  if (props.tasks.length === 0) return {}
  
  const counts = {}
  statusOptions.forEach(opt => counts[opt] = 0)
  props.tasks.forEach(t => {
    if (counts[t.status] !== undefined) counts[t.status]++
  })
  
  const pcts = {}
  Object.keys(counts).forEach(k => {
    pcts[k] = (counts[k] / props.tasks.length * 100).toFixed(1)
  })
  return pcts
})

const priorityPercentages = computed(() => {
  if (props.tasks.length === 0) return {}
  
  const counts = {}
  priorityOptions.forEach(opt => counts[opt] = 0)
  props.tasks.forEach(t => {
    if (counts[t.priority] !== undefined) counts[t.priority]++
  })
  
  const pcts = {}
  Object.keys(counts).forEach(k => {
    pcts[k] = (counts[k] / props.tasks.length * 100).toFixed(1)
  })
  return pcts
})

const typePercentages = computed(() => {
  if (props.tasks.length === 0) return {}
  
  const counts = {}
  typeOptions.forEach(opt => counts[opt] = 0)
  props.tasks.forEach(t => {
    if (counts[t.type] !== undefined) counts[t.type]++
  })
  
  const pcts = {}
  Object.keys(counts).forEach(k => {
    pcts[k] = (counts[k] / props.tasks.length * 100).toFixed(1)
  })
  return pcts
})

const dateColors = ['#d1d5db', '#9ca3af', '#6b7280', '#4b5563']

const datePercentages = computed(() => {
  if (props.tasks.length === 0) return []
  
  const dateCounts = {}
  props.tasks.forEach(t => {
    if (dateCounts[t.date]) {
      dateCounts[t.date]++
    } else {
      dateCounts[t.date] = 1
    }
  })
  
  const result = []
  let colorIdx = 0
  Object.keys(dateCounts).forEach(date => {
    result.push({
      pct: (dateCounts[date] / props.tasks.length * 100).toFixed(1),
      color: dateColors[colorIdx % dateColors.length]
    })
    colorIdx++
  })
  
  return result
})
</script>

<template>
  <div class="rounded-lg overflow-hidden" style="background-color: #2a2d3e;">
    <div class="flex items-start gap-4 p-4">
      <div class="w-1 rounded" style="background-color: #8b5cf6; min-height: 100px;"></div>
      <div class="flex-1">
        <div class="flex items-center gap-2 mb-3">
          <span class="text-gray-400 text-sm">▼</span>
          <span class="text-gray-300 font-medium">All Task</span>
        </div>
        
        <div class="overflow-x-auto">
          <table class="w-full min-w-[800px]">
            <thead>
              <tr style="background-color: #1e2139;">
                <th class="px-3 py-3 text-left text-xs font-semibold text-gray-400 w-10">
                  <input type="checkbox" class="w-4 h-4" />
                </th>
                <th class="px-3 py-3 text-left text-xs font-semibold text-gray-400">Task</th>
                <th class="px-3 py-3 text-left text-xs font-semibold text-gray-400">Developer</th>
                <th class="px-3 py-3 text-left text-xs font-semibold text-gray-400">Status</th>
                <th class="px-3 py-3 text-left text-xs font-semibold text-gray-400">Priority</th>
                <th class="px-3 py-3 text-left text-xs font-semibold text-gray-400">Type</th>
                <th class="px-3 py-3 text-left text-xs font-semibold text-gray-400">Date</th>
                <th class="px-3 py-3 text-left text-xs font-semibold text-gray-400">Estimated SP ⓘ</th>
                <th class="px-3 py-3 text-left text-xs font-semibold text-gray-400">Actual SP ⓘ</th>
                <th class="px-3 py-3 text-left text-xs font-semibold text-gray-400">+</th>
              </tr>
            </thead>
            <tbody>
          <!-- New Task Row -->
          <tr v-if="showNewTaskRow" style="background-color: #1e2139; border: 1px solid #3b82f6;">
            <td class="px-3 py-3">
              <input type="checkbox" class="w-4 h-4" />
            </td>
            <td class="px-3 py-3">
              <input 
                v-model="newTask.task"
                type="text"
                placeholder="Task name..."
                class="w-full px-3 py-2 border border-gray-600 rounded bg-gray-800 text-white text-sm focus:outline-none focus:border-blue-500"
              />
            </td>
            <td class="px-3 sm:px-4 py-3">
              <input 
                :value="Array.isArray(newTask.developers) ? newTask.developers.join(', ') : newTask.developers"
                @input="newTask.developers = $event.target.value.split(',').map(d => d.trim()).filter(d => d)"
                type="text"
                placeholder="John, Jane..."
                class="w-full px-3 py-2 border border-gray-600 rounded bg-gray-800 text-white text-sm focus:outline-none focus:border-blue-500"
              />
            </td>
            <td class="px-3 sm:px-4 py-3">
              <select 
                v-model="newTask.status"
                class="w-full px-3 py-2 border border-gray-600 rounded bg-gray-800 text-white text-sm focus:outline-none focus:border-blue-500"
              >
                <option v-for="opt in statusOptions" :key="opt" :value="opt">{{ opt }}</option>
              </select>
            </td>
            <td class="px-3 sm:px-4 py-3">
              <select 
                v-model="newTask.priority"
                class="w-full px-3 py-2 border border-gray-600 rounded bg-gray-800 text-white text-sm focus:outline-none focus:border-blue-500"
              >
                <option v-for="opt in priorityOptions" :key="opt" :value="opt">{{ opt }}</option>
              </select>
            </td>
            <td class="px-3 sm:px-4 py-3">
              <select 
                v-model="newTask.type"
                class="w-full px-3 py-2 border border-gray-600 rounded bg-gray-800 text-white text-sm focus:outline-none focus:border-blue-500"
              >
                <option v-for="opt in typeOptions" :key="opt" :value="opt">{{ opt }}</option>
              </select>
            </td>
            <td class="px-3 sm:px-4 py-3">
              <input 
                v-model="newTask.date"
                type="date"
                class="w-full px-3 py-2 border border-gray-600 rounded bg-gray-800 text-white text-sm focus:outline-none focus:border-blue-500"
              />
            </td>
            <td class="px-3 sm:px-4 py-3">
              <input 
                v-model="newTask.estimatedSP"
                type="number"
                class="w-full px-3 py-2 border border-gray-600 rounded bg-gray-800 text-white text-sm focus:outline-none focus:border-blue-500"
              />
            </td>
            <td class="px-3 sm:px-4 py-3">
              <input 
                v-model="newTask.actualSP"
                type="number"
                class="w-full px-3 py-2 border border-gray-600 rounded bg-gray-800 text-white text-sm focus:outline-none focus:border-blue-500"
              />
            </td>
            <td class="px-3 sm:px-4 py-3">
              <div class="flex gap-2">
                <button 
                  @click="handleAddTask"
                  class="px-3 py-1.5 bg-blue-600 text-white text-xs sm:text-sm rounded hover:bg-blue-700 font-medium"
                >
                  Add
                </button>
                <button 
                  @click="showNewTaskRow = false"
                  class="px-3 py-1.5 bg-gray-600 text-white text-xs sm:text-sm rounded hover:bg-gray-700 font-medium"
                >
                  Cancel
                </button>
              </div>
            </td>
          </tr>

          <!-- Task Rows -->
          <tr v-for="task in tasks" :key="task.id" style="background-color: #2a2d3e; border-bottom: 1px solid #1e2139;" class="hover:bg-opacity-80 transition-all">
            <td class="px-3 py-3">
              <input type="checkbox" class="w-4 h-4" />
            </td>
            <td class="px-3 py-3" @click="startEdit(task.id, 'task', task.task)">
              <div class="flex items-center gap-2">
                <input 
                  v-if="editingCell === task.id + '-task'"
                  v-model="editingValue"
                  @blur="saveEdit(task, 'task')"
                  @keyup.enter="saveEdit(task, 'task')"
                  @keyup.esc="cancelEdit"
                  class="w-full px-3 py-2 border border-gray-600 rounded bg-gray-800 text-white text-sm focus:outline-none focus:border-blue-500"
                  autofocus
                />
                <span v-else class="cursor-pointer text-gray-300 hover:text-white transition-colors text-sm flex items-center gap-2">
                  {{ task.task }}
                  <svg class="w-4 h-4 text-gray-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z"/>
                  </svg>
                </span>
              </div>
            </td>

            <!-- Developers -->
            <td class="px-3 py-3" @click="startEdit(task.id, 'developers', task.developers)">
              <div class="flex items-center gap-2">
                <input 
                  v-if="editingCell === task.id + '-developers'"
                  v-model="editingValue"
                  @blur="saveEdit(task, 'developers')"
                  @keyup.enter="saveEdit(task, 'developers')"
                  @keyup.esc="cancelEdit"
                  class="w-full px-3 py-2 border border-gray-600 rounded bg-gray-800 text-white text-sm focus:outline-none focus:border-blue-500"
                  placeholder="Comma separated"
                  autofocus
                />
                <span v-else class="cursor-pointer text-gray-300 hover:text-white transition-colors text-sm flex items-center gap-2">
                  <svg class="w-4 h-4 text-gray-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"/>
                  </svg>
                  {{ Array.isArray(task.developers) ? task.developers.join(', ') : task.developers }}
                </span>
              </div>
            </td>

            <!-- Status -->
            <td class="px-3 py-3" @click="startEdit(task.id, 'status', task.status)">
              <select 
                v-if="editingCell === task.id + '-status'"
                v-model="editingValue"
                @change="saveEdit(task, 'status')"
                @blur="cancelEdit"
                class="w-full px-3 py-2 border border-gray-600 rounded bg-gray-800 text-white text-sm focus:outline-none focus:border-blue-500"
                autofocus
              >
                <option v-for="opt in statusOptions" :key="opt" :value="opt">{{ opt }}</option>
              </select>
              <span 
                v-else 
                class="inline-block px-3 py-1 rounded text-white text-xs font-medium cursor-pointer"
                :style="{ backgroundColor: statusColors[task.status] }"
              >
                {{ task.status }}
              </span>
            </td>

            <!-- Priority -->
            <td class="px-3 py-3" @click="startEdit(task.id, 'priority', task.priority)">
              <select 
                v-if="editingCell === task.id + '-priority'"
                v-model="editingValue"
                @change="saveEdit(task, 'priority')"
                @blur="cancelEdit"
                class="w-full px-3 py-2 border border-gray-600 rounded bg-gray-800 text-white text-sm focus:outline-none focus:border-blue-500"
                autofocus
              >
                <option v-for="opt in priorityOptions" :key="opt" :value="opt">{{ opt }}</option>
              </select>
              <span 
                v-else 
                class="inline-block px-3 py-1 rounded text-white text-xs font-medium cursor-pointer"
                :style="{ backgroundColor: priorityColors[task.priority] }"
              >
                {{ task.priority }}
              </span>
            </td>

            <!-- Type -->
            <td class="px-3 py-3" @click="startEdit(task.id, 'type', task.type)">
              <select 
                v-if="editingCell === task.id + '-type'"
                v-model="editingValue"
                @change="saveEdit(task, 'type')"
                @blur="cancelEdit"
                class="w-full px-3 py-2 border border-gray-600 rounded bg-gray-800 text-white text-sm focus:outline-none focus:border-blue-500"
                autofocus
              >
                <option v-for="opt in typeOptions" :key="opt" :value="opt">{{ opt }}</option>
              </select>
              <span 
                v-else 
                class="inline-block px-3 py-1 rounded text-white text-xs font-medium cursor-pointer"
                :style="{ backgroundColor: typeColors[task.type] }"
              >
                {{ task.type }}
              </span>
            </td>

            <!-- Date -->
            <td class="px-3 py-3" @click="startEdit(task.id, 'date', task.date)">
              <input 
                v-if="editingCell === task.id + '-date'"
                v-model="editingValue"
                type="date"
                @blur="saveEdit(task, 'date')"
                @keyup.enter="saveEdit(task, 'date')"
                @keyup.esc="cancelEdit"
                class="w-full px-3 py-2 border border-gray-600 rounded bg-gray-800 text-white text-sm focus:outline-none focus:border-blue-500"
                autofocus
              />
              <span v-else class="cursor-pointer text-gray-400 hover:text-white transition-colors text-sm">{{ formatDate(task.date) }}</span>
            </td>

            <!-- Estimated SP -->
            <td class="px-3 py-3" @click="startEdit(task.id, 'estimatedSP', task.estimatedSP)">
              <input 
                v-if="editingCell === task.id + '-estimatedSP'"
                v-model="editingValue"
                type="number"
                @blur="saveEdit(task, 'estimatedSP')"
                @keyup.enter="saveEdit(task, 'estimatedSP')"
                @keyup.esc="cancelEdit"
                class="w-full px-3 py-2 border border-gray-600 rounded bg-gray-800 text-white text-sm focus:outline-none focus:border-blue-500"
                autofocus
              />
              <span v-else class="cursor-pointer text-gray-400 hover:text-white transition-colors text-sm">{{ task.estimatedSP }} SP</span>
            </td>

            <!-- Actual SP -->
            <td class="px-3 py-3" @click="startEdit(task.id, 'actualSP', task.actualSP)">
              <input 
                v-if="editingCell === task.id + '-actualSP'"
                v-model="editingValue"
                type="number"
                @blur="saveEdit(task, 'actualSP')"
                @keyup.enter="saveEdit(task, 'actualSP')"
                @keyup.esc="cancelEdit"
                class="w-full px-3 py-2 border border-gray-600 rounded bg-gray-800 text-white text-sm focus:outline-none focus:border-blue-500"
                autofocus
              />
              <span v-else class="cursor-pointer text-gray-400 hover:text-white transition-colors text-sm">{{ task.actualSP }} SP</span>
            </td>

            <!-- Actions -->
            <td class="px-3 py-3">
              <button 
                @click="$emit('delete-task', task.id)"
                class="p-2 text-gray-400 hover:text-red-400 transition-colors"
              >
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"/>
                </svg>
              </button>
            </td>
          </tr>

          <tr v-if="!showNewTaskRow" style="background-color: #2a2d3e; border-bottom: 1px solid #1e2139;">
            <td class="px-3 py-3"></td>
            <td colspan="9" class="px-3 py-3">
              <button @click="toggleNewTask" class="text-gray-400 hover:text-white text-sm transition-colors">
                + Add task
              </button>
            </td>
          </tr>

          <tr style="background-color: #1e2139;">
            <td class="px-3 py-3"></td>
            <td class="px-3 py-3"></td>
            <td class="px-3 py-3"></td>
            
            <td class="px-3 py-3">
              <div class="flex h-6 rounded overflow-hidden">
                <div v-for="(pct, status) in statusPercentages" :key="status" 
                     :style="{ width: pct + '%', backgroundColor: statusColors[status] }"
                     v-if="parseFloat(pct) > 0">
                </div>
              </div>
            </td>

            <td class="px-3 py-3">
              <div class="flex h-6 rounded overflow-hidden">
                <div v-for="(pct, priority) in priorityPercentages" :key="priority" 
                     :style="{ width: pct + '%', backgroundColor: priorityColors[priority] }"
                     v-if="parseFloat(pct) > 0">
                </div>
              </div>
            </td>

            <td class="px-3 py-3">
              <div class="flex h-6 rounded overflow-hidden">
                <div v-for="(pct, type) in typePercentages" :key="type" 
                     :style="{ width: pct + '%', backgroundColor: typeColors[type] }"
                     v-if="parseFloat(pct) > 0">
                </div>
              </div>
            </td>

            <td class="px-3 py-3">
              <div class="flex h-6 rounded overflow-hidden">
                <div v-for="(item, idx) in datePercentages" :key="idx" 
                     v-if="item && parseFloat(item.pct) > 0"
                     :style="{ width: item.pct + '%', backgroundColor: item.color }">
                </div>
              </div>
            </td>

            <td class="px-3 py-3">
              <div class="flex h-6 rounded overflow-hidden">
                <div class="w-1/3" style="background-color: #fbbf24;"></div>
                <div class="w-1/3" style="background-color: #f59e0b;"></div>
                <div class="w-1/3" style="background-color: #ea580c;"></div>
              </div>
            </td>

            <td class="px-3 py-3">
              <div class="flex h-6 rounded overflow-hidden">
                <div class="w-1/2" style="background-color: #d1d5db;"></div>
                <div class="w-1/2" style="background-color: #9ca3af;"></div>
              </div>
            </td>

            <td class="px-3 py-3"></td>
          </tr>
        </tbody>
      </table>
    </div>
      </div>
    </div>
  </div>
</template>
