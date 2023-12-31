<script setup>
  import * as mutations from './graphql/mutations';
  import * as queries from './graphql/queries';
  import * as subscriptions from './graphql/subscriptions';
  import { generateClient } from 'aws-amplify/api';
  import { onMounted, ref } from 'vue';

  const name = ref('');
  const description = ref('');
  const todos = ref([]);

  const client = generateClient();

  async function addTodo() {
    if (!name.value || !description.value) return;
    const todo = { name: name.value, description: description.value };
    await client.graphql({
      query: mutations.createTodo,
      variables: { input: todo }
    });
    name.value = '';
    description.value = '';
  }

  async function fetchTodos() {
    const fetchedTodos = await client.graphql({
      query: queries.listTodos
    });

    todos.value = fetchedTodos.data.listTodos.items;
  }

  function subscribeToNewTodos() {
    client
      .graphql({
        query: subscriptions.onCreateTodo
      })
      .subscribe({
        next: ({ data }) => {
          todos.value = [...todos.value, data.onCreateTodo];
        }
      });
  }

  onMounted(() => {
    fetchTodos();
    subscribeToNewTodos();
  });
</script>
<template>
  <div id="app">
    <h1>Todo App</h1>
    <input type="text" v-model="name" placeholder="Todo name" />
    <input type="text" v-model="description" placeholder="Todo description" />
    <button v-on:click="addTodo">Create Todo</button>

    <div v-for="item in todos" :key="item.id">
      <h3>{{ item.name }}</h3>
      <p>{{ item.description }}</p>
    </div>
  </div>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
