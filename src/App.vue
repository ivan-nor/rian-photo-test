<template>
  <div class="addForm">
    <button @click="handleAddProduct" v-if="!isAddProduct">Add New Product</button>
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
          return { backgroundColor: 'LightCoral' }
        case 'develop':
          return { backgroundColor: 'Gold' }
        case 'done':
          return { backgroundColor: 'MediumAquamarine' }
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
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  max-width: 1000px;
  padding: 10px;
}

.column {
  flex: 0 0 calc(33.33%);
  display: flex;
  flex-direction: column;
  gap: 20px;
  width: 200px; /* Фиксированная ширина для колонки */
  padding: 20px;
  background-color: #3498db;
  border: 1px solid #000;
  border-radius: 5px;
  min-height: 800px;
}

body { margin: 0; }
</style>
