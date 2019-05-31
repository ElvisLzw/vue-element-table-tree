<template>
  <div id="app">
    <table-tree/>
  </div>
</template>

<script>
import TableTree from './components/TableTree.vue'

export default {
  name: 'app',
  components: {
    TableTree
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
