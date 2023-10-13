<template>
  <div class="addForm">
    <button class="addButton" @click="handleAddProduct" v-if="!isAddProduct">Add New Product</button>
      <form v-if="isAddProduct">
        <h3>Добавить товар:</h3>

        <label for="inputTitle">Название:</label>
        <input type="text" v-model="inputTitle">

        <label for="inputCategory">Категория:</label>
        <input type="text" v-model="inputCategory">

        <label for="inputDescription">Описание:</label>
        <textarea v-model="inputDescription" rows="5"></textarea>

        <label for="editedPrice">Цена:</label>
        <input type="number" min="0" v-model="inputPrice">

        <button @click="handleSaveProduct">Save</button>
        <button @click="cancelAddingProduct">Cancel</button>
      </form>
  </div>
  <div class="grid-container">
    <div class="column drop-zone"
      v-for="key in Object.keys(productsState)"
      :key="key"
      :style="getColumnStyle(key)"
      @drop="ondrop($event, key)"
      @dragover.prevent
      @dragenter.prevent
    >
      <h1>{{ key }}</h1>
      <div v-for="item in getListProducts(key)" :key="item.id">
        <CardComponent
          :status="key"
          :item="item"
          @saveChanges="handleEditCard"
          @setStatus="handleSetStatus"
          draggable="true"
          @dragstart="ondragstart($event, item, key)"
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
      },
      isAddProduct: false, // добавить добавоенпе карточки
      inputDescription: '',
      inputPrice: null,
      inputTitle: '',
      inputCategory: '',
      counterId: null
    }
  },
  async created () { // выделить в отдельную функцию обновления, запуск и при создании, и после добавления задачи
    const products = await this.getProducts() // запрос на сервер за продуктами
    const sliced = products.slice(0, 3) // для разработки беру только несколько
    this.productsState.unprocessed = sliced.map((p) => p.id) // все продукты кладем в стейт, все айди в необработанные
    this.products = this.sortedOnRating(sliced)
    console.log('CREATE APP', sliced, sliced.map((p) => p.id))

    this.counterId = Math.max(...this.products.map(p => p.id)) + 1
    console.log('counterID', this.counterId)
  },
  methods: { // сделать событие добавления карточки
    async getProducts () { // здесь возможно в конце  добавить функционал обновления после добавления новой карточки продукта
      return await fetch('https://fakestoreapi.com/products')
        .then((res) => res.json())
        .then((products) => { // переделать добавление карточек
          // const prevProductIdies = [...this.productsState.unprocessed, ...this.productsState.develop, ...this.productsState.done]
          // console.log('GET PROD prev products idies', prevProductIdies)
          return products
        })
    },
    ondragstart (evt, item, status) {
      console.log('CARD dregStart', item, status)
      evt.dataTransfer.setData('productId', item.id)
      evt.dataTransfer.setData('prevStatus', status)
      evt.dataTransfer.dropEffect = 'move'
      evt.dataTransfer.effectAllowed = 'move'
    },
    ondrop (evt, nextStatus) {
      const productId = evt.dataTransfer.getData('productId')
      const prevStatus = evt.dataTransfer.getData('prevStatus')
      console.log('ON DROP !!!', productId, prevStatus, nextStatus)
      this.moveProduct(productId, prevStatus, nextStatus)
    },
    moveProduct (productId, prevStatus, nextStatus) {
      this.productsState[prevStatus] = [...this.productsState[prevStatus].filter((id) => id !== +productId)]
      this.productsState[nextStatus] = [...this.productsState[nextStatus], +productId]

      console.log('in App.vue, card id: ', typeof productId, prevStatus, this.productsState[prevStatus], this.productsState[nextStatus])
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
    handleSetStatus (productId, prevStatus) {
      const nextStatus = (prevStatus === 'unprocessed') ? 'develop' : 'done'
      this.moveProduct(productId, prevStatus, nextStatus)
    },
    handleAddProduct () {
      this.isAddProduct = true
    },
    cancelAddingProduct () {
      this.isAddProduct = false
    },
    sortedOnRating (arr) { // сортируем каждый раз когда идет обновление
      return arr.sort((a, b) => (a.rating.rate < b.rating.rate) ? 1 : -1)
    },
    getColumnStyle (status) {
      switch (status) {
        case 'unprocessed':
          return { backgroundColor: 'Lavender' }
        case 'develop':
          return { backgroundColor: 'PaleGoldenrod' }
        case 'done':
          return { backgroundColor: 'Aquamarine' }
        default:
          return { backgroundColor: 'gray' }
      }
    },
    getListProducts (status) {
      const filtered = this.products.filter((p) => this.productsState[status].includes(p.id))
      const sorted = this.sortedOnRating(filtered)
      return sorted
    }
  },
  watch: {
    // productsState () { // отразить новую задачу в tasks
    //   console.log('WATCH products state', this.productsState)
    // },
    // products () {
    //   console.log('WATCH products', this.products)
    // },
    'productsState.unprocessed' () {
      console.log('WATCH unprocessed', this.productsState.unprocessed)
    },
    'productsState.develop' () {
      console.log('WATCH develop', this.productsState.develop)
    },
    'productsState.done' () {
      console.log('WATCH done', this.productsState.done)
    }
  }
}
</script>

<style>
.grid-container {
  display: flex;
  gap: 20px;
  max-width: 1000px;
  padding: 10px;
}

.column {
  flex-basis: 30%;
  display: flex;
  flex-direction: column;
  gap: 20px;
  min-width: 200px; /* Фиксированная ширина для колонки */
  padding: 20px;
  background-color: #3498db;
  box-shadow: 0 4px 4px rgba(0, 0, 0, 0.2);
  border: 0px solid #000;
  border-radius: 5px;
  min-height: 800px;
  align-items: flex-start;
}

.addButton {
  background-color: #ad3288;
  color: #fff;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
  font-weight: bold;
  font-family: Arial, sans-serif;
  border-radius: 10px;
  justify-content: center;
}

.addForm {
  max-width: 1000px;
  max-height: 100%;
  overflow-y: auto;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  border-radius: 10px; /* Закругленные углы для формы */
  display: flex;
  flex-direction: column;
  align-items: justify;
  text-align: justify;
  margin: 10px 10px;
  color: #333;
  font-family: Arial, sans-serif;
}

h1 {
  margin: 0;
}

body {
  margin: 0;
  font-family: Arial, sans-serif;
}
</style>
