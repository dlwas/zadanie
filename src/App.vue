<template>
  <div id="app" class="container-fluid container-sm">
    <div>
      <h1>Zadnie rekrutacyjne dla Olicom</h1>
    </div>
    <div>
      <Table
        :endpoint="this.endpoint"
        :search="this.search"
        :sorting="this.sorting"
        :pagination="this.pagination"
        dataTypes="id,name,email,company,website"
      />
    </div>
  </div>
</template>

<script>
import Table from './components/Table.vue'

export default {
  name: 'App',
  components: {
    Table
  },
  data() {
    return {
      endpoint: 'https://jsonplaceholder.typicode.com/users',
      search: true,
      sorting: true,
      pagination: false,
      dataTypes: ''
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* text-align: center; */
  color: #2c3e50;
  margin-top: 60px;
}
</style>
