<script setup>
import { computed, ref } from 'vue';

import AppTabbar from '../components/AppTabbar.vue';
import TaskCard from '../components/TaskCard.vue';
import TaskCounter from '../components/TaskCounter.vue';

import {
  taskState,
  setTaskStatus,
  setTaskPriority,
} from '../store/tasks.js';

import '../css/task-components.css';

const totalTasks = computed(() => taskState.tasks.length);

const completedTasks = computed(() =>
  taskState.tasks.filter(task => task.status === 'done').length
);

const progressTasks = computed(() =>
  taskState.tasks.filter(task => task.status === 'progress').length
);

const plannedTasks = computed(() =>
  taskState.tasks.filter(task => task.status === 'todo').length
);

const completionPercentage = computed(() =>
  totalTasks.value
    ? Math.round((completedTasks.value / totalTasks.value) * 100)
    : 0
);

const activeTasks = computed(() => {
  const order = { high: 0, medium: 1, low: 2 };

  return taskState.tasks
    .filter(task => task.status !== 'done')
    .slice()
    .sort((a, b) => order[a.priority] - order[b.priority])
    .slice(0, 3);
});

const selectedTask = ref(null);

function handleStatusChange(payload) {
  setTaskStatus(payload);
}

function handlePriorityChange(payload) {
  setTaskPriority(payload);
}

function handleTaskSelect(payload) {
  selectedTask.value = payload;
}
</script>

<template>
  <f7-page name="home" class="tf3-page">
    <f7-navbar title="TaskFlow">
      <f7-nav-right>
        <f7-link href="/about/">Про застосунок</f7-link>
      </f7-nav-right>
    </f7-navbar>

    <div class="tf3-container">
      <section class="tf3-hero">
        <div class="tf3-eyebrow">TASKFLOW / DASHBOARD</div>
        <h1>Панель керування</h1>
        <p>Контролюйте виконання завдань та їхні пріоритети.</p>
      </section>

      <section class="tf3-stat-grid tf3-stat-grid--four">
        <TaskCounter
          :count="totalTasks"
          label="Усього завдань"
        />

        <TaskCounter
          :count="progressTasks"
          label="У роботі"
        />

        <TaskCounter
          :count="completedTasks"
          label="Виконано"
        />

        <TaskCounter
          :count="plannedTasks"
          label="Заплановано"
        />
      </section>

      <section class="tf3-progress-section">
        <div class="tf3-progress-heading">
          <span>Прогрес виконання</span>
          <strong>{{ completionPercentage }}%</strong>
        </div>

        <div class="tf3-progress-track">
          <div
            class="tf3-progress-fill"
            :style="{ width: completionPercentage + '%' }"
          ></div>
        </div>
      </section>

      <section class="tf3-section">
        <div class="tf3-section-heading">
          <div>
            <div class="tf3-eyebrow">PRIORITY TASKS</div>
            <h2>Актуальні завдання</h2>
          </div>

          <f7-button href="/tasks/" outline small>
            Переглянути всі
          </f7-button>
        </div>

        <div v-if="selectedTask" class="tf3-selected-task">
          Вибране завдання: {{ selectedTask.title }}
        </div>

        <div v-if="activeTasks.length" class="tf3-task-grid">
          <TaskCard
            v-for="task in activeTasks"
            :key="task.id"
            :task="task"
            @status-change="handleStatusChange"
            @priority-change="handlePriorityChange"
            @task-select="handleTaskSelect"
          />
        </div>

        <div v-else class="tf3-empty">
          Наразі всі завдання виконані.
        </div>
      </section>
    </div>

    <AppTabbar active="home" />
  </f7-page>
</template>