<script setup>
import { computed, nextTick, ref } from 'vue';

import AppTabbar from '../components/AppTabbar.vue';
import TaskCard from '../components/TaskCard.vue';
import TaskForm from '../components/TaskForm.vue';

import {
  taskState,
  addTask,
  updateTask,
  setTaskStatus,
  setTaskPriority,
} from '../store/tasks.js';

import '../css/task-components.css';
import '../css/task-form.css';

// Фільтри
const selectedStatus = ref('all');
const selectedPriority = ref('all');

// Стан форми
const formVisible = ref(false);
const editingId = ref(null);
const formAnchor = ref(null);
const feedback = ref('');

// Вибране завдання з попередньої практичної
const selectedTask = ref(null);

// Варіанти фільтрації за статусом
const statusOptions = [
  { value: 'all', label: 'Усі' },
  { value: 'todo', label: 'Заплановано' },
  { value: 'progress', label: 'У роботі' },
  { value: 'done', label: 'Виконано' },
];

// Варіанти фільтрації за пріоритетом
const priorityOptions = [
  { value: 'all', label: 'Усі' },
  { value: 'low', label: 'Низький' },
  { value: 'medium', label: 'Середній' },
  { value: 'high', label: 'Високий' },
];

// Фільтрація та сортування завдань
const filteredTasks = computed(() => {
  const order = {
    high: 0,
    medium: 1,
    low: 2,
  };

  return taskState.tasks
    .filter((task) => (
      (selectedStatus.value === 'all' ||
        task.status === selectedStatus.value) &&
      (selectedPriority.value === 'all' ||
        task.priority === selectedPriority.value)
    ))
    .slice()
    .sort((a, b) => order[a.priority] - order[b.priority]);
});

// Завдання, яке редагується
const editingTask = computed(() => (
  taskState.tasks.find(
    (task) => task.id === editingId.value
  ) ?? null
));

// Обробники зміни статусу та пріоритету
function handleStatusChange(payload) {
  setTaskStatus(payload);
}

function handlePriorityChange(payload) {
  setTaskPriority(payload);
}

// Обробник вибору завдання
function handleTaskSelect(payload) {
  selectedTask.value = payload;
}

// Прокручування сторінки до форми
async function scrollToForm() {
  await nextTick();

  formAnchor.value?.scrollIntoView({
    behavior: 'smooth',
    block: 'start',
  });
}

// Відкриття порожньої форми
async function createTask() {
  editingId.value = null;
  formVisible.value = true;
  feedback.value = '';

  await scrollToForm();
}

// Відкриття форми редагування
async function editTask(id) {
  editingId.value = id;
  formVisible.value = true;
  feedback.value = '';

  await scrollToForm();
}

// Закриття форми
function cancelForm() {
  formVisible.value = false;
  editingId.value = null;
}

// Збереження нового або відредагованого завдання
function saveTask(data) {
  const isEditing = editingId.value !== null;

  const saved = isEditing
    ? updateTask({
        id: editingId.value,
        ...data,
      })
    : addTask(data);

  if (!saved) {
    feedback.value =
      'Не вдалося зберегти завдання. Перевірте дані форми.';
    return;
  }

  feedback.value = isEditing
    ? `Завдання «${saved.title}» оновлено.`
    : `Завдання «${saved.title}» створено.`;

  cancelForm();

  // Після збереження скидаємо фільтри
  selectedStatus.value = 'all';
  selectedPriority.value = 'all';
}
</script>

<template>
  <f7-page name="tasks" class="tf3-page">
    <f7-navbar title="Мої завдання" />

    <div class="tf3-container">

      <!-- Заголовок сторінки -->
      <section class="tf3-hero">
        <div class="tf3-eyebrow">
          TASKFLOW / TASK MANAGER
        </div>

        <h1>Мої завдання</h1>

        <p>
          Керуйте завданнями, статусами та пріоритетами.
        </p>

        <button
          type="button"
          class="tf4-new-task"
          @click="createTask"
        >
          + Додати завдання
        </button>
      </section>

      <!-- Повідомлення про результат -->
      <p
        v-if="feedback"
        class="tf4-feedback"
        role="status"
      >
        {{ feedback }}
      </p>

      <!-- Форма створення або редагування -->
      <section
        v-if="formVisible"
        ref="formAnchor"
        class="tf4-form-section"
      >
        <TaskForm
          :key="editingId ?? 'new'"
          :task="editingTask"
          @save="saveTask"
          @cancel="cancelForm"
        />
      </section>

      <!-- Вибране завдання -->
      <div
        v-if="selectedTask"
        class="tf3-selected-task"
      >
        Вибране завдання: {{ selectedTask.title }}
      </div>

      <!-- Фільтр за статусом і пріоритетом -->
      <section class="tf3-filters">

        <div class="tf3-filter-group">
          <div class="tf3-filter-title">
            Фільтр за статусом
          </div>

          <div class="tf3-filter-buttons">
            <button
              v-for="option in statusOptions"
              :key="option.value"
              type="button"
              class="tf3-filter-button"
              :class="{
                'is-active': selectedStatus === option.value
              }"
              :aria-pressed="selectedStatus === option.value"
              @click="selectedStatus = option.value"
            >
              {{ option.label }}
            </button>
          </div>
        </div>

        <div class="tf3-filter-group">
          <div class="tf3-filter-title">
            Фільтр за пріоритетом
          </div>

          <div class="tf3-filter-buttons">
            <button
              v-for="option in priorityOptions"
              :key="option.value"
              type="button"
              class="tf3-filter-button"
              :class="{
                'is-active': selectedPriority === option.value
              }"
              :aria-pressed="selectedPriority === option.value"
              @click="selectedPriority = option.value"
            >
              {{ option.label }}
            </button>
          </div>
        </div>

      </section>

      <!-- Список завдань -->
      <section class="tf3-section">

        <div class="tf3-section-heading">
          <div>
            <div class="tf3-eyebrow">
              TASK LIST
            </div>

            <h2>Список завдань</h2>
          </div>

          <div class="tf3-counter">
            Знайдено: {{ filteredTasks.length }}
          </div>
        </div>

        <div
          v-if="filteredTasks.length"
          class="tf3-task-grid"
        >
          <TaskCard
            v-for="task in filteredTasks"
            :key="task.id"
            :task="task"
            :editable="true"
            @status-change="handleStatusChange"
            @priority-change="handlePriorityChange"
            @task-select="handleTaskSelect"
            @edit="editTask"
          />
        </div>

        <div
          v-else
          class="tf3-empty"
        >
          За обраними фільтрами завдань не знайдено.
        </div>

      </section>
    </div>

    <AppTabbar active="tasks" />
  </f7-page>
</template>