
<script setup>
import { ref, computed } from 'vue'

const tasks = ref([])
const inputTask = ref('')
const searchTask = ref('')
const activeFilter = ref('semua')
const isDarkMode = ref(false)
const editingId = ref(null)
const editInput = ref('')

const totalTasks = computed(() => tasks.value.length)

const completedTasks = computed(() =>
  tasks.value.filter(task => task.status).length
)

const pendingTasks = computed(() =>
  tasks.value.filter(task => !task.status).length
)

const progress = computed(() => {
  if (totalTasks.value === 0) return 0
  return Math.round((completedTasks.value / totalTasks.value) * 100)
})

const filteredTasks = computed(() => {
  let result = tasks.value

  if (activeFilter.value === 'selesai') {
    result = result.filter(task => task.status)
  } else if (activeFilter.value === 'belum') {
    result = result.filter(task => !task.status)
  }

  if (searchTask.value.trim()) {
    result = result.filter(task =>
      task.title.toLowerCase().includes(searchTask.value.trim().toLowerCase())
    )
  }

  return result
})

const addTask = () => {
  if (!inputTask.value.trim()) return

  tasks.value.push({
    id: Date.now(),
    title: inputTask.value.trim(),
    status: false
  })

  inputTask.value = ''
}

const deleteTask = (id) => {
  if (confirm('Apakah kamu yakin ingin menghapus tugas ini?')) {
    tasks.value = tasks.value.filter(task => task.id !== id)
  }
}

const startEdit = (task) => {
  editingId.value = task.id
  editInput.value = task.title
}

const saveEdit = (id) => {
  if (!editInput.value.trim()) {
    alert('Nama tugas tidak boleh kosong!')
    return
  }

  const task = tasks.value.find(item => item.id === id)

  if (task) {
    task.title = editInput.value.trim()
  }

  cancelEdit()
}

const cancelEdit = () => {
  editingId.value = null
  editInput.value = ''
}
</script>

<template>
  <section class="app-container" :class="{ dark: isDarkMode }">
    <div class="task-card">

      <div class="top-header">
        <h1 class="title">
          Taskify <span>Vue</span>
        </h1>

        <button
          class="theme-button"
          @click="isDarkMode = !isDarkMode"
          :aria-label="isDarkMode ? 'Aktifkan mode terang' : 'Aktifkan mode gelap'"
        >
          {{ isDarkMode ? '☀️' : '🌙' }}
        </button>
      </div>

      <!-- Tambah Tugas -->
      <form @submit.prevent="addTask" class="task-form">
        <input
          v-model="inputTask"
          type="text"
          placeholder="Masukkan Tugas Baru"
          class="task-input"
        />

        <button type="submit" class="add-button">
          Tambah
        </button>
      </form>

      <!-- Daftar Tugas -->
      <h3 class="section-title">Daftar Tugas</h3>

      <ul class="task-list" v-auto-animate>
        <li
          v-for="task in filteredTasks"
          :key="task.id"
          class="task-item"
        >
          <template v-if="editingId === task.id">
            <input
              v-model="editInput"
              class="edit-input"
              @keyup.enter="saveEdit(task.id)"
              @keyup.esc="cancelEdit"
              autofocus
            />

            <div class="action-buttons">
              <button
                @click="saveEdit(task.id)"
                class="save-button"
              >
                Simpan
              </button>

              <button
                @click="cancelEdit"
                class="cancel-button"
              >
                Batal
              </button>
            </div>
          </template>

          <template v-else>
            <div class="task-content">
              <input
                type="checkbox"
                v-model="task.status"
                class="task-checkbox"
              />

              <span :class="{ completed: task.status }">
                {{ task.title }}
              </span>
            </div>

            <div class="action-buttons">
              <button
                @click="startEdit(task)"
                class="edit-button"
              >
                Edit
              </button>

              <button
                @click="deleteTask(task.id)"
                class="delete-button"
              >
                Hapus
              </button>
            </div>
          </template>
        </li>
      </ul>

      <p v-if="filteredTasks.length === 0" class="empty-message">
        {{ totalTasks === 0
          ? 'Belum ada tugas. Yuk, tambahkan tugas!'
          : 'Tugas tidak ditemukan.' }}
      </p>

      <!-- Pencarian -->
      <div class="search-section">
        <input
          v-model="searchTask"
          type="text"
          placeholder="Cari tugas..."
          class="search-input"
        />
      </div>

      <!-- Filter -->
      <div class="filter-section">
        <button
          @click="activeFilter = 'semua'"
          :class="{ active: activeFilter === 'semua' }"
          class="filter-button"
        >
          Semua
        </button>

        <button
          @click="activeFilter = 'belum'"
          :class="{ active: activeFilter === 'belum' }"
          class="filter-button"
        >
          Belum Selesai
        </button>

        <button
          @click="activeFilter = 'selesai'"
          :class="{ active: activeFilter === 'selesai' }"
          class="filter-button"
        >
          Selesai
        </button>
      </div>

      <!-- Progress Bar -->
      <div class="progress-section">
        <div class="progress-header">
          <span>Progress Tugas</span>
          <strong>{{ progress }}%</strong>
        </div>

        <div class="progress-track">
          <div
            class="progress-fill"
            :style="{ width: progress + '%' }"
          ></div>
        </div>

        <p class="progress-info">
          {{ completedTasks }} dari {{ totalTasks }} tugas selesai
        </p>
      </div>

      <!-- Statistik -->
      <div class="statistics">
        <div class="stat-box">
          <strong>{{ totalTasks }}</strong>
          <span>Total</span>
        </div>

        <div class="stat-box">
          <strong>{{ completedTasks }}</strong>
          <span>Selesai</span>
        </div>

        <div class="stat-box">
          <strong>{{ pendingTasks }}</strong>
          <span>Belum Selesai</span>
        </div>
      </div>

    </div>
  </section>
</template>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  min-height: 100vh;
  font-family: Arial, Helvetica, sans-serif;
}

.app-container {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 65px 20px;
  background: linear-gradient(
    120deg,
    #a9d9ff,
    #cbb8ff,
    #ffbfdf,
    #ffe0a8
  );
  transition: background 0.3s ease;
}

.task-card {
  width: 100%;
  max-width: 440px;
  padding: 24px 25px 28px;
  border-radius: 23px;
  background: #edf2ff;
  box-shadow: none;
  transition: background 0.3s ease;
}

.top-header {
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  margin-bottom: 23px;
}

.title {
  text-align: center;
  font-size: 29px;
  font-weight: 800;
  color: #28346f;
  margin: 0;
}

.title span {
  color: #397cff;
}

.theme-button {
  position: absolute;
  right: 0;
  width: 38px;
  height: 38px;
  border: none;
  border-radius: 12px;
  background: #e1eaff;
  font-size: 18px;
  cursor: pointer;
}

.task-form {
  display: flex;
  gap: 10px;
  margin-bottom: 22px;
}

.task-input,
.search-input,
.edit-input {
  min-width: 0;
  height: 48px;
  padding: 0 13px;
  font-size: 13px;
  color: #26365e;
  border: 1.5px solid #c8d9ff;
  outline: none;
  border-radius: 12px;
  background: #edf2ff;
  box-shadow:
    inset 3px 3px 6px #cbd5ec,
    inset -3px -3px 6px #ffffff;
}

.task-input {
  flex: 1;
}

.task-input::placeholder,
.search-input::placeholder {
  color: #8996b4;
}

.task-input:focus,
.search-input:focus,
.edit-input:focus {
  border-color: #8eb5ff;
}

.add-button {
  width: 85px;
  height: 48px;
  border: none;
  border-radius: 12px;
  background: linear-gradient(135deg, #50b5ff, #5967f5);
  color: white;
  font-size: 13px;
  font-weight: bold;
  cursor: pointer;
  box-shadow:
    3px 3px 8px #b4c3ed,
    -3px -3px 7px #ffffff;
  transition: 0.2s;
}

.add-button:hover {
  transform: translateY(-2px);
  filter: brightness(1.08);
}

.section-title {
  color: #28346f;
  font-size: 15px;
  margin: 0 0 13px;
}

/* Daftar tugas */
.task-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
  padding: 0;
  margin: 0;
  list-style: none;
}

.task-item {
  min-height: 53px;
  padding: 10px 13px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  border-radius: 12px;
  background: #f4f5ff;
  box-shadow:
    3px 3px 8px #cbd3e9,
    -3px -3px 8px #ffffff;
}

.task-item:nth-child(4n + 1) {
  background: #fce9f5;
}

.task-item:nth-child(4n + 2) {
  background: #e5f2ff;
}

.task-item:nth-child(4n + 3) {
  background: #e6f8f0;
}

.task-item:nth-child(4n + 4) {
  background: #fff1df;
}

.task-content {
  display: flex;
  align-items: center;
  gap: 10px;
  min-width: 0;
  color: #28365f;
  font-size: 14px;
  overflow-wrap: anywhere;
}

.task-checkbox {
  width: 15px;
  height: 15px;
  accent-color: #6678f7;
  cursor: pointer;
  flex-shrink: 0;
}

.completed {
  text-decoration: line-through;
  color: #9ba7c0;
}

.action-buttons {
  display: flex;
  gap: 5px;
  flex-shrink: 0;
}

.edit-button,
.delete-button,
.save-button,
.cancel-button {
  border: none;
  font-size: 11px;
  font-weight: 600;
  cursor: pointer;
  padding: 6px 9px;
  border-radius: 8px;
  transition: 0.2s;
}

.edit-button {
  background: #dceaff;
  color: #397cff;
}

.delete-button {
  background: #ffe0eb;
  color: #f34e7b;
}

.save-button {
  background: #d9f6e7;
  color: #238653;
}

.cancel-button {
  background: #e5e9f2;
  color: #637399;
}

.edit-button:hover,
.delete-button:hover,
.save-button:hover,
.cancel-button:hover {
  filter: brightness(0.95);
}

.edit-input {
  flex: 1;
  width: 100%;
  height: 36px;
}

.empty-message {
  text-align: center;
  color: #8996b4;
  font-size: 12px;
  margin: 18px 0;
}

/* Pencarian */
.search-section {
  margin-top: 20px;
}

.search-input {
  width: 100%;
  height: 42px;
}

/* Filter */
.filter-section {
  display: flex;
  gap: 7px;
  margin-top: 14px;
  margin-bottom: 20px;
}

.filter-button {
  flex: 1;
  padding: 9px 5px;
  border: none;
  border-radius: 9px;
  background: #e5eaff;
  color: #637399;
  font-size: 11px;
  font-weight: 600;
  cursor: pointer;
  transition: 0.2s;
}

.filter-button.active {
  background: linear-gradient(135deg, #50b5ff, #5967f5);
  color: white;
}

/* Progress */
.progress-section {
  margin-bottom: 20px;
}

.progress-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #28346f;
  font-size: 13px;
  margin-bottom: 9px;
}

.progress-header strong {
  color: #397cff;
}

.progress-track {
  width: 100%;
  height: 12px;
  overflow: hidden;
  border-radius: 20px;
  background: #dce5fa;
  box-shadow:
    inset 2px 2px 4px #c5cfe4,
    inset -2px -2px 4px #ffffff;
}

.progress-fill {
  height: 100%;
  border-radius: 20px;
  background: linear-gradient(90deg, #50b5ff, #5967f5, #c18bff);
  transition: width 0.35s ease;
}

.progress-info {
  color: #8996b4;
  font-size: 11px;
  margin: 8px 0 0;
}

/* Statistik */
.statistics {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 9px;
  margin-bottom: 5px;
}

.stat-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 5px;
  min-height: 65px;
  padding: 9px 4px;
  border-radius: 12px;
  background: #e5edff;
  box-shadow:
    3px 3px 7px #cbd5ec,
    -3px -3px 7px #ffffff;
}

.stat-box strong {
  font-size: 19px;
  color: #397cff;
}

.stat-box span {
  font-size: 10px;
  color: #657394;
  text-align: center;
}

/* Dark Mode */
.app-container.dark {
  background: linear-gradient(
    120deg,
    #171d35,
    #252442,
    #34283f,
    #292d43
  );
}

.dark .task-card {
  background: #252b43;
}

.dark .title,
.dark .section-title,
.dark .progress-header {
  color: #edf2ff;
}

.dark .title span,
.dark .progress-header strong {
  color: #8ebaff;
}

.dark .theme-button {
  background: #39415f;
}

.dark .task-input,
.dark .search-input,
.dark .edit-input {
  background: #303954;
  color: #edf2ff;
  border-color: #485577;
  box-shadow:
    inset 3px 3px 6px #20283d,
    inset -3px -3px 6px #3c4663;
}

.dark .task-input::placeholder,
.dark .search-input::placeholder {
  color: #a0acc7;
}

.dark .task-item,
.dark .task-item:nth-child(n) {
  background: #303954;
  box-shadow:
    3px 3px 8px #1c2338,
    -3px -3px 8px #343d59;
}

.dark .task-content {
  color: #edf2ff;
}

.dark .completed {
  color: #9ba7c0;
}

.dark .filter-button {
  background: #39415f;
  color: #c5cee5;
}

.dark .filter-button.active {
  background: linear-gradient(135deg, #50b5ff, #5967f5);
  color: white;
}

.dark .stat-box {
  background: #303954;
  box-shadow:
    3px 3px 7px #1c2338,
    -3px -3px 7px #343d59;
}

.dark .stat-box strong {
  color: #8ebaff;
}

.dark .stat-box span,
.dark .progress-info,
.dark .empty-message {
  color: #b2bdd5;
}

.dark .progress-track {
  background: #1d2740;
  box-shadow: inset 2px 2px 4px #151c30;
}

/* Tampilan HP */
@media (max-width: 480px) {
  .app-container {
    padding: 35px 15px;
  }

  .task-card {
    max-width: 100%;
    padding: 22px 18px;
  }

  .title {
    font-size: 26px;
  }

  .task-input {
    font-size: 12px;
  }

  .task-content {
    font-size: 13px;
  }

  .filter-button {
    font-size: 10px;
  }

  .task-item {
    padding: 9px;
  }
}
</style>
