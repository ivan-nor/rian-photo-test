<template>
  <div class="grid-container">
      <ColumnComponent v-for="key in Object.keys(tasks)"
        :key="key"
        :status="key"
        :products="products"
        :idiesOfTasks="tasks[key]"
      />
  </div>
</template>

<script>
import ColumnComponent from './components/ColumnComponent.vue'

export default {
  name: 'App',
  components: {
    ColumnComponent
  },
  data () {
    return {
      products: [], // { 'id', 'title', 'price', 'description', 'category', 'image', 'rating' }
      tasks: {
        unprocessed: [], // массивы с айдишниками
        // develop: [77, 76],
        // done: [55, 54]
      },
      unprocessed: [], // массивы с айдишниками
      develop: [],
      done: []
    }
  },

  async created () { // выделить в отдельную функцию обновления, запуск и при создании, и после добавления задачи
    await fetch('https://fakestoreapi.com/products')
      .then(res => res.json())
      .then(json => {
        this.products = json.slice(0, 3)

        const responseIdies = this.products.map(({ id }) => id)
        console.log('RESPONSE', this.products, responseIdies)

        // const oldTaskIdies = [...this.unprocessed, ...this.develop, ...this.done]
        // console.log(oldTaskIdies)

        this.tasks = { ...this.tasks, unprocessed: responseIdies }
      })
  },
  watch: {
    tasks () { // отразить новую задачу в tasks.unprocessed
      console.log('WATCH unprocessed')
    },
    products () {
      // console.log('WTAHC products', this.products)
    }
  }
  // сделать событие добавления карточки
}
</script>

<style>
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr); /* Три колонки с равной шириной */
  gap: 10px; /* Расстояние между колонками */
}
</style>
