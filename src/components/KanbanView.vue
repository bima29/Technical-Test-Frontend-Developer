<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  tasks: Array
})

const emit = defineEmits(['add-task', 'update-task'])

const showModal = ref(false)
const showCommentModal = ref(false)
const currentCommentTask = ref(null)
const newComment = ref('')
const draggedTask = ref(null)
const newTask = ref({
  task: '',
  developers: '',
  status: 'Ready to start',
  priority: 'Medium',
  type: 'Feature Enhancements',
  date: '',
  estimatedSP: 0,
  actualSP: 0,
  comments: []
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
  'Critical': '#ef4444',
  'High': '#3b82f6',
  'Medium': '#06b6d4',
  'Low': '#10b981',
  'Best Effort': '#ec4899'
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

const statusPercentages = computed(() => {
  if (props.tasks.length === 0) return {}
  
  const counts = {}
  statusColumns.forEach(opt => counts[opt] = 0)
  props.tasks.forEach(t => {
    if (counts[t.status] !== undefined) counts[t.status]++
  })
  
  const pcts = {}
  Object.keys(counts).forEach(k => {
    pcts[k] = (counts[k] / props.tasks.length * 100).toFixed(1)
  })
  return pcts
})

const getTasksByStatus = (status) => {
  let filtered = [];
  for (let i = 0; i < props.tasks.length; i++) {
    if ((props.tasks[i].status === status)) {
      filtered.push(props.tasks[i]);
    }
  }
  return filtered;
};

const openModal = () => {
  if (!showModal.value) {
    showModal.value = true;
  } else {
    showModal.value = true;
  }
};

const closeModal = () => {
  if ((showModal.value === true)) {
    showModal.value = false;
  } else {
    showModal.value = false;
  }
  resetForm();
};

const resetForm = () => {
  const empty = {
    task: '',
    developers: '',
    status: 'Ready to start',
    priority: 'Medium',
    type: 'Feature Enhancements',
    date: '',
    estimatedSP: 0,
    actualSP: 0,
    comments: []
  };
  newTask.value = empty;
};

const openCommentModal = (task) => {
  currentCommentTask.value = task;
  if (!showCommentModal.value) {
    showCommentModal.value = true;
  } else {
    showCommentModal.value = true;
  }
};

const addComment = () => {
  if (!currentCommentTask.value) return;
  const text = (newComment.value || '').trim();
  if (!text) return;
  const updated = {
    ...currentCommentTask.value,
    comments: [
      ...((currentCommentTask.value.comments) || []),
      { text, createdAt: new Date().toISOString() }
    ]
  };
  emit('update-task', updated);
  currentCommentTask.value = updated;
  newComment.value = '';
};

const getDeveloperInitials = (developers) => {
  if ((!developers) || (developers.length === 0)) return '?';
  return developers[0].charAt(0).toUpperCase();
};

const getDevInitial = (name) => {
  if ((!name) || (typeof name !== 'string')) return '?';
  return name.charAt(0).toUpperCase();
};

const handleSubmit = () => {
  const name = newTask.value.task;
  if ((!name) || (name === '')) {
    alert('Task name is required!');
    return;
  }

  let taskData = { ...newTask.value };
  const devLine = newTask.value.developers;
  let devs = [];
  if (devLine && typeof devLine === 'string') {
    const parts = devLine.split(',');
    for (let i = 0; i < parts.length; i++) {
      const p = parts[i];
      const s = p.trim();
      if (s) devs.push(s);
    }
  }
  taskData.developers = devs;

  emit('add-task', taskData);
  closeModal();
};

const handleDragStart = (task) => {
  draggedTask.value = task;
};

const handleDragOver = (e) => {
  let evt = e;
  e.preventDefault();
  evt = null;
};

const handleDrop = (e, newStatus) => {
  e.preventDefault();
  
  if ((draggedTask.value) && (draggedTask.value.status !== newStatus)) {
    let updatedTask = { ...draggedTask.value };
    const targetStatus = newStatus;
    if (targetStatus) {
      updatedTask.status = targetStatus;
    } else {
      updatedTask.status = newStatus;
    }
    const payload = updatedTask;
    emit('update-task', payload);
  }
  
  draggedTask.value = null;
};

const handleDragEnd = () => {
  draggedTask.value = null;
};

defineExpose({ openModal })
</script>

<template>
  <div>
    <div class="flex h-2 rounded overflow-hidden mb-6">
      <div v-for="(pct, status) in statusPercentages" :key="status" 
           :style="{ width: pct + '%', backgroundColor: statusBgColors[status] }"
           v-if="parseFloat(pct) > 0">
      </div>
    </div>

    <div class="overflow-x-auto">
      <div class="flex gap-4 min-w-max">
    <div 
      v-for="status in statusColumns" 
      :key="status"
      class="rounded-lg overflow-hidden w-72 flex-shrink-0"
      @dragover="handleDragOver"
      @drop="handleDrop($event, status)"
    >
      <div class="p-3 text-white font-semibold text-sm flex items-center justify-between rounded-t-lg"
           :style="{ backgroundColor: statusBgColors[status] }">
        <span>{{ status }} {{ getTasksByStatus(status).length }}</span>
      </div>

      <div class="p-3 space-y-3 min-h-[400px] rounded-b-lg" style="background-color: #2a2d3e;">
        <div 
          v-for="task in getTasksByStatus(status)" 
          :key="task.id"
          class="rounded-lg p-4 cursor-move transition-all hover:shadow-lg"
          style="background-color: #3a3d52;"
          draggable="true"
          @dragstart="handleDragStart(task)"
          @dragend="handleDragEnd"
        >
          <h4 class="text-gray-200 mb-3 text-sm font-medium leading-snug">{{ task.task }}</h4>

          <div class="flex items-center gap-2 mb-2">
            <span 
              class="inline-block px-2 py-1 rounded text-white text-xs font-medium"
              :style="{ backgroundColor: priorityColors[task.priority] }"
            >
              {{ task.priority }}
            </span>
            
            <span class="text-xs text-gray-400">{{ task.estimatedSP }} SP</span>
          </div>

          <div class="mb-3">
            <span 
              class="inline-block px-2 py-1 rounded text-white text-xs"
              :style="{ backgroundColor: typeColors[task.type] }"
            >
              {{ task.type }}
            </span>
          </div>

          <div class="flex items-center gap-2 pt-2 border-t border-gray-600">
            <div class="flex -space-x-2">
              <div 
                v-for="(dev, i) in (Array.isArray(task.developers) ? task.developers : [])" 
                :key="i"
                class="w-7 h-7 rounded-full bg-blue-500 flex items-center justify-center text-xs text-white font-semibold ring-2 ring-[#2a2d3e]"
                :title="dev"
              >
                {{ getDevInitial(dev) }}
              </div>
            </div>
            <button @click="openCommentModal(task)" class="p-1 text-gray-400 hover:text-gray-200 transition-colors">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z"/>
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
    </div>
  </div>

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

  <div v-if="showCommentModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50" @click.self="showCommentModal = false">
    <div class="rounded-lg p-6 w-full max-w-lg mx-4" style="background-color: #2a2d3e;">
      <h3 class="text-white font-bold mb-4">Comments for: {{ currentCommentTask?.task }}</h3>
      <div class="mb-4 p-4 bg-gray-800 rounded min-h-[150px] space-y-3">
        <div v-if="currentCommentTask?.comments && currentCommentTask.comments.length > 0" class="space-y-2">
          <div v-for="(c, idx) in currentCommentTask.comments" :key="idx" class="text-sm text-gray-300">
            <div class="flex items-center gap-2">
              <span class="text-gray-200">•</span>
              <span>{{ c.text }}</span>
            </div>
            <div class="text-xs text-gray-500 ml-4" v-if="c.createdAt">{{ new Date(c.createdAt).toLocaleString() }}</div>
          </div>
        </div>
        <p v-else class="text-gray-400 text-sm">No comments yet...</p>
      </div>
      <textarea 
        v-model="newComment"
        placeholder="Add a comment..."
        class="w-full px-4 py-2 rounded bg-gray-800 text-white border border-gray-600 focus:outline-none focus:border-blue-500 mb-4"
        rows="3"
      ></textarea>
      <div class="flex gap-2">
        <button @click="addComment" class="px-4 py-2 bg-green-600 text-white rounded hover:bg-green-700">Add Comment</button>
        <button @click="showCommentModal = false" class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700">
          Close
        </button>
      </div>
    </div>
  </div>
</template>
