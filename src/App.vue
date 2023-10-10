<template>
  <div class="grid-container">
      <ColumnComponent v-for="key in Object.keys(productsState)"
        :key="key"
        :status="key"
        :products="products"
        :idiesOfProducts="productsState[key]"
        @saveChanges="handleEditCard"
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
      productsState: {
        unprocessed: [], // массивы с айдишниками
        develop: [],
        done: []
      }
    }
  },

  async created () { // выделить в отдельную функцию обновления, запуск и при создании, и после добавления задачи
    const products = await this.getProducts()
    const sliced = products.slice(0, 10)
    console.log('CREATE APP', sliced, sliced.map((p) => p.id))
    this.productsState.unprocessed = sliced.map((p) => p.id)
    this.products = this.sortedOnRating(sliced)
  },
  methods: {
    async getProducts () {
      return await fetch('https://fakestoreapi.com/products')
        .then((res) => res.json())
        .then((products) => {
          // const prevProductIdies = [...this.productsState.unprocessed, ...this.productsState.develop, ...this.productsState.done]
          // console.log('GET PROD prev products idies', prevProductIdies)

          return products
        })
    },
    // Метод для обработки события edit-card
    handleEditCard (cardId, editedDescription, editedPrice) {
      // Обновите состояние или выполните другие действия на основе события
      console.log('Событие saveChanges получено в App.vue', cardId, editedDescription, editedPrice)
      // Например, обновите products или productsState в соответствии с изменениями
      let [currentProduct] = this.products.filter(({ id }) => id === cardId)
      const productsWithoutCurrent = this.products.filter(({ id }) => id !== cardId)
      console.log('curr prod', currentProduct, currentProduct.id)
      currentProduct = { ...currentProduct, description: editedDescription, price: editedPrice }
      const newProducts = [...productsWithoutCurrent, currentProduct]
      this.products = this.sortedOnRating(newProducts)
    },
    sortedOnRating (arr) {
      console.log('SORTED before', arr)
      const sorted = arr.sort((a, b) => (a.rating.rate < b.rating.rate) ? 1 : -1)
      console.log('SORTED after', sorted)
      return sorted
    }
  },
  watch: {
    productsState () { // отразить новую задачу в tasks
      console.log('WATCH products state', this.productsState)
    },
    products () {
      console.log('WATCH products', this.products)
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
