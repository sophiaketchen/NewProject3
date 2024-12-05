<script setup>
import { ref, computed, watchEffect } from 'vue'

const STORAGE_KEY = 'vue-todomvc'

// filter functions for different todo states
const filters = {
  all: (todos) => todos,
  active: (todos) => todos.filter((todo) => !todo.completed),
  completed: (todos) => todos.filter((todo) => todo.completed)
}

// state
const todos = ref(JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'))
const visibility = ref('all')
const editedTodo = ref()

// derived state
const filteredTodos = computed(() => filters[visibility.value](todos.value))
const remaining = computed(() => filters.active(todos.value).length)

// handle routing
window.addEventListener('hashchange', onHashChange)
onHashChange()

// persist state
watchEffect(() => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(todos.value))
})

// toggle the completion state of all todos (check/uncheck all)
function toggleAll(e) {
  todos.value.forEach((todo) => (todo.completed = e.target.checked))
}

// Add a new todo when the user presses Enter in the input field
function addTodo(e) {
  const value = e.target.value.trim()
  if (value) {
    todos.value.push({
      id: Date.now(),
      title: value,
      completed: false
    })
    e.target.value = ''
  }
}

// Remove a specific todo from the list
function removeTodo(todo) {
  todos.value.splice(todos.value.indexOf(todo), 1)
}

// State to hold the original title of a todo before editing
let beforeEditCache = ''
function editTodo(todo) {
  beforeEditCache = todo.title
  editedTodo.value = todo
}

// Cancel the editing of a todo and restore its original title
function cancelEdit(todo) {
  editedTodo.value = null
  todo.title = beforeEditCache
}

// Finish editing a todo
function doneEdit(todo) {
  if (editedTodo.value) {
    editedTodo.value = null
    todo.title = todo.title.trim()
    if (!todo.title) removeTodo(todo)
  }
}

// Remove all completed todos
function removeCompleted() {
  todos.value = filters.active(todos.value)
}

// Handle changes in URL hash to change visibility filter
function onHashChange() {
  const route = window.location.hash.replace(/#\/?/, '')
  if (filters[route]) {
    visibility.value = route
  } else {
    window.location.hash = ''
    visibility.value = 'all'
  }
}
</script>

<template>
  <section class="todoapp">
        <!-- Header with todo input -->
    <header class="header">
      <h1>To Do:</h1>
        <!-- Input field for adding new todos -->
      <input
        class="new-todo"
        autofocus
        placeholder="What needs to be done?"
        @keyup.enter="addTodo"
      >
    </header>
        <!-- Main section with todo list, only show if there are todos -->
    <section class="main" v-show="todos.length">
        <!-- Checkbox to mark all todos as complete -->
      <input
        id="toggle-all"
        class="toggle-all"
        type="checkbox"
        :checked="remaining === 0"
        @change="toggleAll"
      >
      <label for="toggle-all">Mark all as complete</label>
        <!-- List of todos -->
      <ul class="todo-list">
        <li
          v-for="todo in filteredTodos"
          class="todo"
          :key="todo.id"
          :class="{ completed: todo.completed, editing: todo === editedTodo }"
        >
          <div class="view">
            <!-- Checkbox to toggle completion of individual todo -->
            <input class="toggle" type="checkbox" v-model="todo.completed">
            <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
            <button class="destroy" @click="removeTodo(todo)"></button>
          </div>
           <!-- Input field for editing todo title -->
          <input
            v-if="todo === editedTodo"
            class="edit"
            type="text"
            v-model="todo.title"
            @vue:mounted="({ el }) => el.focus()"
            @blur="doneEdit(todo)"
            @keyup.enter="doneEdit(todo)"
            @keyup.escape="cancelEdit(todo)"
          >
        </li>
      </ul>
    </section>
    <!-- Footer with todo count and filters -->
    <footer class="footer" v-show="todos.length">
      <span class="todo-count">
        <strong>{{ remaining }}</strong>
        <span>{{ remaining === 1 ? ' item' : ' items' }} left</span>
      </span>
      <ul class="filters">
      <!-- Links for visibility filters (all, active, completed) -->
        <li>
          <a href="#/all" :class="{ selected: visibility === 'all' }">All</a>
        </li>
        <li>
          <a href="#/active" :class="{ selected: visibility === 'active' }">Active</a>
        </li>
        <li>
          <a href="#/completed" :class="{ selected: visibility === 'completed' }">Completed</a>
        </li>
      </ul>
      <!-- Button to clear completed todos -->
      <button class="clear-completed" @click="removeCompleted" v-show="todos.length > remaining">
        Clear completed
      </button>
    </footer>
  </section>
</template>

<style scoped>
@import '@/assets/styles/todoapp.css'; 
</style>


