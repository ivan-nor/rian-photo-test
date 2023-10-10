<template>
  <div v-for="task in tasks" :key="task.id">
    <!-- Добавьте больше карточек для первой колонки -->
    <div class="card">{{ task }}</div>
  </div>
</template>

<script>
export default {
  name: 'ColumnComponent',
  props: {
    idiesOfTasks: Array,
    products: Array,
    status: String
  },
  data () {
    return {
    }
  },
  created () {
    console.log('CREAATE Column', this.status, this.idiesOfTasks)
  },
  computed: {
    tasks () {
      console.log('COLUMN computed tasks', this.idiesOfTasks, this.idiesOfTasks.includes(12))
      return this.products.filter((task) => this.idiesOfTasks.includes(task.id))
    }
  },
  updated () {
    console.log('COLUMN tasks', this.tasks)
  }
}
</script>

<style>
  .card {
    width: 100%;
    height: 150px; /* Фиксированный размер карточки */
    background-color: #f2f2f2;
    border: 1px solid #ddd;
    padding: 10px;
  }
</style>
