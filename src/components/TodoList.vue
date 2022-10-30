<template>
  <div class="todo-list">
    <input
      class="todo-input"
      type="text"
      placeholder="Add a new task"
      aria-label="Add a new task"
      v-model="newTodo"
      @keyup.enter="addTodo"
    />
    <div class="filters">
      <div>
        <button
          class="filter-button"
          :class="{ active: filter === 'all' }"
          @click="filter = 'all'"
        >
          All
        </button>
        <button
          class="filter-button"
          :class="{ active: filter === 'active' }"
          @click="filter = 'active'"
        >
          Active
        </button>
        <button
          class="filter-button"
          :class="{ active: filter === 'completed' }"
          @click="filter = 'completed'"
        >
          Completed
        </button>
      </div>
      <Transition name="fade">
        <button
          class="clear-all-button"
          v-if="showClearCompletedBtn()"
          @click="clearCompleted"
        >
          Clear completed
        </button>
      </Transition>
    </div>
    <TransitionGroup name="fade">
      <div
        class="todo-item"
        v-for="(todo, index) in todosFilter"
        :key="todo.id"
      >
        <input
          type="checkbox"
          aria-label="Complete todo"
          v-model="todo.completed"
        />
        <div
          class="todo-label"
          v-if="!todo.editing"
          @dblclick="editTodo(todo)"
          :class="{ completed: todo.completed }"
        >
          {{ todo.title }}
        </div>
        <input
          class="todo-edit"
          type="text"
          v-model="todo.title"
          v-else
          @keyup.enter="doneEdit(todo)"
          @blur="doneEdit(todo)"
          @keyup.escape="cancleEdit(todo)"
          v-focus
        />
        <button
          class="remove-item"
          aria-label="Remove todo"
          @click="removeTodo(index)"
        >
          &times;
        </button>
      </div>
    </TransitionGroup>
    <div class="controls">
      <div>
        <input
          type="checkbox"
          name="check-all"
          id="check-all"
          :checked="noRemainingTodos"
          @change="checkAllTodos"
        />
        <label for="check-all" class="check-all-label">Check all</label>
      </div>
      <p>Remaining: {{ todosRemaining }}</p>
    </div>
  </div>
</template>

<script>
import { v4 as uuidv4 } from 'uuid';

export default {
  name: 'todo-list',
  data() {
    return {
      newTodo: '',
      beforeEditCache: '',
      filter: 'all',
      todos: [
        {
          id: uuidv4(),
          title: 'Build todo app',
          completed: false,
          editing: false,
        },
        {
          id: uuidv4(),
          title: 'Add todo',
          completed: false,
          editing: false,
        },
        {
          id: uuidv4(),
          title: 'Complete todo',
          completed: true,
          editing: false,
        },
      ],
    };
  },
  computed: {
    todosRemaining() {
      return this.todos.filter((todo) => !todo.completed).length;
    },
    noRemainingTodos() {
      return this.todosRemaining === 0;
    },
    todosFilter() {
      if (this.filter === 'active')
        return this.todos.filter((todo) => !todo.completed);
      if (this.filter === 'completed')
        return this.todos.filter((todo) => todo.completed);
      return this.todos;
    },
  },
  directives: {
    focus: {
      mounted: (el) => el.focus(),
    },
  },
  methods: {
    addTodo() {
      if (this.newTodo.trim() === '') return;
      this.todos.push({
        id: uuidv4(),
        title: this.newTodo,
        completed: false,
      });
    },
    removeTodo(index) {
      this.todos.splice(index, 1);
    },
    editTodo(todo) {
      this.beforeEditCache = todo.title;
      todo.editing = true;
      console.log(todo.id);
    },
    cancleEdit(todo) {
      todo.title = this.beforeEditCache;
      todo.editing = false;
    },
    doneEdit(todo) {
      if (todo.title.trim() === '') todo.title = this.beforeEditCache;
      todo.editing = false;
    },
    checkAllTodos() {
      this.todos.forEach((todo) => (todo.completed = event.target.checked));
    },
    clearCompleted() {
      this.todos = this.todos.filter((todo) => !todo.completed);
    },
    showClearCompletedBtn() {
      return this.todos.filter((todo) => todo.completed).length > 0;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.todo-list {
  max-width: 600px;
  margin: 0 auto;
  padding: 30px;
  background-color: #fff;
  border-radius: 8px;
}

.todo-input {
  width: 100%;
  padding: 10px;
  background-image: url('../assets/list-icon.svg');
  background-repeat: no-repeat;
  background-size: 20px;
  background-position: 10px center;
  border: 1px solid #000;
  border-radius: 8px 0 8px 0;
  padding-left: 40px;
  transition: border-radius 0.5s;
}

.todo-input:active,
.todo-input:focus {
  border-radius: 0 8px 0 8px;
}

input[type='checkbox'] {
  accent-color: rgb(9, 9, 121);
}

.todo-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  text-align: left;
  margin: 10px 0;
}

.todo-label,
.todo-edit {
  width: 100%;
  padding: 10px;
  margin: 0 5px;
  color: rgb(2, 0, 36);
  font-size: 16px;
}

.remove-item {
  background-color: inherit;
  border: none;
  cursor: pointer;
}

.remove-item:hover {
  color: rgb(193, 24, 24);
}

.completed {
  text-decoration: line-through;
}

.controls {
  display: flex;
  justify-content: space-between;
  border-top: 1px solid #000;
  padding: 10px 0;
  text-align: left;
}

.check-all-label {
  padding: 10px;
}

.filters {
  display: flex;
  justify-content: space-between;
  margin: 20px 0;
}

.filter-button {
  background-color: inherit;
  border: none;
  border-radius: 4px;
  margin-right: 10px;
  padding: 5px;
  color: rgb(2, 0, 36);
}

.active {
  background-image: linear-gradient(
    to right,
    rgb(127, 255, 127),
    rgb(26, 204, 26)
  );
}

.clear-all-button {
  background-image: linear-gradient(
    to right,
    rgb(224, 62, 62),
    rgb(193, 24, 24)
  );
  border: none;
  border-radius: 4px;
  color: #fff;
  padding: 5px;
}

.clear-all-button:hover {
  filter: brightness(110%);
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
