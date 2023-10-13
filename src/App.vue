<template>
  <div >
    <div class="addForm">
      <button class="addButton" @click="handleClickAddingForm" v-if="!isAddProduct">Добавить товар</button>
      <div v-if="isAddProduct" class="form-container">
        <h3>Добавить товар</h3>

        <label for="inputTitle">Название:</label>
        <input type="text" v-model="form.title" name="inputTitle">

        <label for="inputCategory">Категория:</label>
        <input type="text" v-model="form.category" name="inputCategory">

        <label for="inputDescription">Описание:</label>
        <textarea v-model="form.description" name="inputDescription"></textarea>

        <label for="inputPrice">Цена:</label>
        <input type="number" min="0" v-model="form.price" name="inputPrice">

        <button class="addProductButton" @click="handleAddProduct">Save</button>
        <button class="cancelAddingProductButton" @click="cancelAddingProduct">Cancel</button>
      </div>
    </div>
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
          @deleteProduct="handleDeleteCard"
          @changeStatus="handleChangeStatus"
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
      isAddProduct: false, // добавить добавленiе карточки
      form: {
        description: '',
        price: null,
        title: '',
        category: '',
        image: null
      }
    }
  },
  created () {
    this.updateProducts()
  },
  methods: {
    updateProducts () {
      this.getProducts()
    },
    async getProducts () {
      return await fetch('https://fakestoreapi.com/products')
        .then((res) => res.json())
        .then((products) => {
          const prevProductIdies = [...this.productsState.unprocessed, ...this.productsState.develop, ...this.productsState.done]
          const newProductsIdies = products.map((p) => p.id).filter((id) => !prevProductIdies.includes(id))
          this.productsState.unprocessed = [...this.productsState.unprocessed, ...newProductsIdies]
          this.products = this.sortedOnRating(products.slice(0, 3))
        })
    },
    async addProduct (product) {
      console.log('product', product)
      await fetch('https://fakestoreapi.com/products', { method: 'POST', body: JSON.stringify(product) })
      this.getProducts()
    },
    async editProduct (product) {
      await fetch(`https://fakestoreapi.com/products/${product.id}`, { method: 'PATCH', body: JSON.stringify(product) })
      this.getProducts()
    },
    ondragstart (evt, item, status) {
      evt.dataTransfer.setData('productId', item.id)
      evt.dataTransfer.setData('prevStatus', status)
      evt.dataTransfer.dropEffect = 'move'
      evt.dataTransfer.effectAllowed = 'move'
    },
    ondrop (evt, nextStatus) {
      const productId = evt.dataTransfer.getData('productId')
      const prevStatus = evt.dataTransfer.getData('prevStatus')
      this.moveProduct(productId, prevStatus, nextStatus)
    },
    moveProduct (productId, prevStatus, nextStatus) {
      this.productsState[prevStatus] = [...this.productsState[prevStatus].filter((id) => id !== +productId)]
      this.productsState[nextStatus] = [...this.productsState[nextStatus], +productId]
    },
    handleEditCard (editedProduct) {
      this.editProduct(editedProduct)
    },
    async handleDeleteCard (product, status) {
      console.log('APP delete card', product.id)
      this.products = this.products.filter((p) => p.id !== product.id)
      this.productsState[status] = this.productsState[status].filter((id) => id !== product.id)
      await fetch(`https://fakestoreapi.com/products/${product.id}`, { method: 'DELETE' })
    },
    handleChangeStatus (productId, prevStatus) {
      const nextStatus = (prevStatus === 'unprocessed') ? 'develop' : 'done'
      this.moveProduct(productId, prevStatus, nextStatus)
    },
    handleClickAddingForm () {
      this.isAddProduct = true
    },
    handleAddProduct () {
      console.log('APP adding product')
      this.isAddProduct = false
      this.addProduct({ ...this.form })
      this.form = {
        description: '',
        price: null,
        title: '',
        category: '',
        image: null
      }
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
  }
}
</script>

<style>
body {
  margin: 0;
  font-family: "Gill Sans", sans-serif;
  background-color: #ebebeb;
}

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
  box-shadow: 0 4px 4px rgba(0, 0, 0, 0.2);
  border-radius: 5px;
  min-height: 800px;
  align-items: center;
}

.column h1 {
  margin: 0;
  text-align: center;
  color: #767c81;
}

.addButton {
  background-color: goldenrod;
  color: #fff;
  border: none;
  padding: 10px 20px;
  cursor: pointer;
  font-weight: bold;
  border-radius: 10px;
  text-align: center;
  width: 300px;
}

button:hover {
  background-color: blue;
}

.addForm {
  max-height: 100%;
  overflow-y: auto;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: justify;
  margin: 10px 10px;
  color: #333;
  padding: 10px 10px ;
  background-color: #f9f9f9;
}

.form-container {
  width: 300px;
  display: flex;
  flex-direction: column;
  gap: 5px;
  padding: 20px;
  background-color: #f9f9f9;
}

.form-container h3 {
  margin: 0;
  text-align: center;
}
</style>
