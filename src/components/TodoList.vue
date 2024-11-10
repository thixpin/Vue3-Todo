<template>
  <div class="min-h-screen bg-gray-100 flex items-center justify-center">
    <div class="bg-white p-6 rounded-lg shadow-lg w-96">
      <h1 class="text-2xl font-semibold text-center mb-6">Todo App</h1>

      <!-- New Todo Input -->
      <div class="mb-4 flex items-center">
        <input type="text" v-model="newTodo" @keyup.enter="addTodo"
          class="w-full px-4 py-2 border rounded-l-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
          placeholder="Add a new todo" />
        <button @click="addTodo"
          class="px-4 py-2 bg-blue-500 text-white rounded-r-lg hover:bg-blue-600 focus:outline-none">
          Add
        </button>
      </div>

      <!-- Todo List -->
      <ul class="space-y-3">
        <li v-for="todo in todos" :key="todo.id"
          class="flex items-center justify-between p-4 border border-gray-300 rounded-lg hover:bg-gray-50">

          <div class="flex items-center space-x-4" @click="toggleComplete(todo)">
            <span :class="{
              'h-5 w-5 border-2 border-gray-400 rounded-full flex items-center justify-center': true,
              'bg-blue-500': todo.completed,
              'bg-white': !todo.completed
            }" class="cursor-pointer">
            </span>

            <!-- Task text -->
            <span :class="{
              'line-through text-gray-400': todo.completed,
              'text-gray-800': !todo.completed
            }" class="text-lg cursor-pointer">
              {{ todo.text }}
            </span>
          </div>


          <button @click="deleteTodo(todo.id)" class="ml-4 text-red-500 hover:text-red-700 focus:outline-none">
            Delete
          </button>
        </li>
      </ul>
    </div>
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
        this.todos = snapshot.docs
                      .map(doc => ({ id: doc.id, ...doc.data() }))
                      .sort((a, b) => b.createdAt - a.createdAt);
      });
    },
    async addTodo() {
      if (this.newTodo.trim()) {
        const todosCollection = collection(db, 'todos');
        await addDoc(todosCollection, {
          text: this.newTodo,
          completed: false,
          createdAt: new Date(),
          completedAt: null,
        });
        this.newTodo = ''; // Clear input after adding
      }
    },
    async toggleComplete(todo) {
      const todoRef = doc(db, 'todos', todo.id);
      await updateDoc(todoRef, {
        completed: !todo.completed,
        completedAt: todo.completed ? null : new Date(),
      });
    },
    async deleteTodo(id) {
      const todoRef = doc(db, 'todos', id);
      await deleteDoc(todoRef);
    },
  },
};
</script>

<style scoped>
/* Custom Styles */
</style>
