<template>
  <div class="flex-container">
    <div class="column" v-for="key in Object.keys(tasks)" :key="key">
    <!--   <UnprocessedComponent  :unprocessed="unprocessed" :products="products" />
      <DevelopComponent :develop="develop" :products="products" />
      <DoneComponent :done="done" :products="products" />  -->

      <ColumnComponent  :status="key" :products="products" :idiesOfTasks="tasks[key]"/>
    </div>
  </div>
</template>

<script>
// import UnprocessedComponent from './components/UnprocessedComponent.vue'
// import DevelopComponent from './components/DevelopComponent.vue'
// import DoneComponent from './components/DoneComponent.vue'
import ColumnComponent from './components/ColumnComponent.vue'

export default {
  name: 'App',
  components: {
    // UnprocessedComponent,
    // DevelopComponent,
    // DoneComponent,
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
        this.products = json

        const responseIdies = json.map(({ id }) => id)
        console.log('RESPONSE', json, responseIdies)

        const oldTaskIdies = [...this.unprocessed, ...this.develop, ...this.done]
        console.log(oldTaskIdies)

        this.unprocessed = responseIdies
        this.tasks.unprocessed = responseIdies
      })
  },
  watch: {
    tasks () { // отразить новую задачу в tasks.unprocessed
      console.log('WATCH unprocessed')
    },
    products () {
      console.log('WTAHC products', this.products)
    }
  }
  // сделать событие добавления карточки
}
</script>

<style>
.flex-container {
  display: flex;
  justify-content: space-between; /* Распределяет колонки равномерно по горизонтали */
  gap: 20px; /* Расстояние между карточками */
}

.column {
  flex-basis: calc(33.33% - 20px); /* Ширина каждой колонки, вычитая расстояние */
  display: flex;
  flex-direction: column;
}
</style>
