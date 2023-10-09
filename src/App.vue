<template>
  <div class="column">
    <UnprocessedComponent  :unprocessed="unprocessed" :products="products" />
    <DevelopComponent :develop="develop" :products="products" />
    <DoneComponent :done="done" :products="products" />
  </div>
</template>

<script>
import UnprocessedComponent from './components/UnprocessedComponent.vue'
import DevelopComponent from './components/DevelopComponent.vue'
import DoneComponent from './components/DoneComponent.vue'

export default {
  name: 'App',
  components: {
    UnprocessedComponent,
    DevelopComponent,
    DoneComponent
  },
  data () {
    return {
      products: [], // { 'id', 'title', 'price', 'description', 'category', 'image', 'rating' }
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
      })
  },
  watch: {
    unprocessed () { // отразить новую задачу в tasks.unprocessed
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

</style>
