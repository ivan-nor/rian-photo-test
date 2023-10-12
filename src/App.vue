<template>
  <div class="grid-container">
      <div class="column" v-for="key in Object.keys(productsState)" :key="key" :style="getColumnStyle(key)">
        <div v-for="item in getListProducts(key)" :key="item.id">
          <CardComponent
            :status="key"
            :item="item"
            @saveChanges="handleEditCard"
            @setStatus="handleSetStatus"
          />
        </div>
  </div>
  </div>
</template>

<script>
import CardComponent from './components/CardComponent.vue'

export default {
  name: 'App',
  components: {
    CardComponent
  },
  data () {
    return {
      products: [], // { 'id', 'title', 'price', 'description', 'category', 'image', 'rating' }
      productsState: {
        unprocessed: [],
        develop: [],
        done: []
      }
    }
  },
  async created () { // выделить в отдельную функцию обновления, запуск и при создании, и после добавления задачи
    const products = await this.getProducts() // запрос на сервер за продуктами
    const sliced = products.slice(0, 3) // для разработки беру только несколько
    this.productsState.unprocessed = sliced.map((p) => p.id) // все продукты кладем в стейт, все айди в необработанные
    this.products = this.sortedOnRating(sliced)
    console.log('CREATE APP', sliced, sliced.map((p) => p.id))
  },
  methods: { // сделать событие добавления карточки
    async getProducts () { // здесь возможно в конце  добавить функционал обновления после добавления новой карточки продукта
      return await fetch('https://fakestoreapi.com/products')
        .then((res) => res.json())
        .then((products) => {
          // const prevProductIdies = [...this.productsState.unprocessed, ...this.productsState.develop, ...this.productsState.done]
          // console.log('GET PROD prev products idies', prevProductIdies)
          return products
        })
    },
    handleEditCard (cardId, editedDescription, editedPrice) { // Метод для обработки события edit-card
      console.log('Событие saveChanges получено в App.vue', cardId, editedDescription, editedPrice)

      let [currentProduct] = this.products.filter(({ id }) => id === cardId)
      const productsWithoutCurrent = this.products.filter(({ id }) => id !== cardId)
      console.log('curr prod', currentProduct, currentProduct.id)
      currentProduct = { ...currentProduct, description: editedDescription, price: editedPrice }
      const newProducts = [...productsWithoutCurrent, currentProduct]
      this.products = this.sortedOnRating(newProducts)
    },
    handleSetStatus (cardId, prevStatus) {
      const getNextStatus = (currStatus) => (currStatus === 'unprocessed') ? 'develop' : 'done'

      this.productsState[prevStatus] = [...this.productsState[prevStatus].filter((id) => id !== cardId)]
      this.productsState[getNextStatus(prevStatus)] = [...this.productsState[getNextStatus(prevStatus)], cardId]

      console.log('in App.vue, card id: ', cardId, prevStatus, getNextStatus(prevStatus), this.productsState[prevStatus], this.productsState[getNextStatus(prevStatus)])
    },
    sortedOnRating (arr) { // сортируем каждый раз когда идет обновление
      return arr.sort((a, b) => (a.rating.rate < b.rating.rate) ? 1 : -1)
    },
    getColumnStyle (status) {
      switch (status) {
        case 'unprocessed':
          return { backgroundColor: 'red' }
        case 'develop':
          return { backgroundColor: 'yellow' }
        case 'done':
          return { backgroundColor: 'green' }
        default:
          return { backgroundColor: 'gray' }
      }
    },
    getListProducts (status) {
      return this.products.filter((p) => this.productsState[status].includes(p.id))
    }
  },
  watch: {
    // productsState () { // отразить новую задачу в tasks
    //   console.log('WATCH products state', this.productsState)
    // },
    // products () {
    //   console.log('WATCH products', this.products)
    // },
    // 'productsState.unprocessed' () {
    //   console.log('WATCH unprocessed', this.productsState.unprocessed)
    // },
    // 'productsState.develop' () {
    //   console.log('WATCH develop', this.productsState.develop)
    // },
    // 'productsState.done' () {
    //   console.log('WATCH done', this.productsState.done)
    // }
  }
}
</script>

<style>
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr); /* Три колонки с равной шириной */
  gap: 10px; /* Расстояние между колонками */
}

.column {
  display: flex;
  flex-direction: column;
  gap: 10px; /* Расстояние между карточками внутри колонки */
  max-width: 300px; /* Максимальная ширина колонки */
  background-color: #3498db;
  padding: 10px;
}
</style>
