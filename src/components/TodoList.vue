<template>
  <div class="todo-app">
    <h1>TODO List</h1>
    <input v-model="newTodo" @keyup.enter="addTodo" placeholder="Add a new task" />
    <ul>
      <li v-for="todo in todos" :key="todo.id">
        <span :class="{ done: todo.completed }" @click="toggleComplete(todo)">
          {{ todo.text }}
        </span>
        <button @click="deleteTodo(todo.id)">Delete</button>
      </li>
    </ul>
  </div>
</template>

<script>
import { db } from '../firebase';
import { collection, onSnapshot, addDoc, updateDoc, deleteDoc, doc } from 'firebase/firestore';

export default {
  data() {
    return {
      todos: [],
      newTodo: '',
    };
  },
  created() {
    this.listenForUpdates();
  },
  methods: {
    // Listen for real-time updates in the 'todos' collection
    listenForUpdates() {
      const todosCollection = collection(db, 'todos');
      onSnapshot(todosCollection, (snapshot) => {
        this.todos = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      });
    },
    async addTodo() {
      if (this.newTodo.trim()) {
        const todosCollection = collection(db, 'todos');
        await addDoc(todosCollection, {
          text: this.newTodo,
          completed: false,
        });
        this.newTodo = ''; // Clear input after adding
      }
    },
    async toggleComplete(todo) {
      const todoRef = doc(db, 'todos', todo.id);
      await updateDoc(todoRef, {
        completed: !todo.completed,
      });
    },
    async deleteTodo(id) {
      const todoRef = doc(db, 'todos', id);
      await deleteDoc(todoRef);
    },
  },
};


</script>

<style>
.todo-app {
  max-width: 400px;
  margin: auto;
  text-align: center;
}

.done {
  text-decoration: line-through;
}
</style>