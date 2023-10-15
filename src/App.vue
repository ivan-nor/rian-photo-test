<template>
  <div >
    <div class="add-form">
      <button class="add-button" @click="handleClickAddingForm" v-if="!isAddingProduct">Добавить товар</button>
      <div v-if="isAddingProduct" class="form-container">
        <h3>Добавить товар</h3>

        <label for="inputTitle">Название:</label>
        <input type="text" v-model="form.title" name="inputTitle">

        <label for="inputCategory">Категория:</label>
        <input type="text" v-model="form.category" name="inputCategory">

        <label for="inputDescription">Описание:</label>
        <textarea v-model="form.description" name="inputDescription"></textarea>

        <label for="inputPrice">Цена:</label>
        <input type="number" min="0" v-model.number="form.price" name="inputPrice">

        <button class="edit-button" type="submit" @click="handleAddProduct">Save</button>
        <button class="delete-button" @click="handleClickAddingForm">Cancel</button>
      </div>
    </div>
  </div>
  <div class="grid-container">
    <div class="column drop-zone"
      v-for="status in Object.keys(productsState)"
      :key="status"
      :style="getColumnStyle(status)"
      @drop="ondrop($event, status)"
      @dragover.prevent
      @dragenter.prevent
    >
      <h1>{{ status }}</h1>
      <div v-for="item in getListProducts(status)" :key="item.id">
        <CardComponent
          :status="status"
          :item="item"
          @editProduct="handleEditProduct"
          @deleteProduct="handleDeleteProduct"
          @changeStatus="handleChangeStatus"
          @dragstart="ondragstart($event, item, status)"
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
      products: [],
      productsState: {
        unprocessed: [],
        develop: [],
        done: []
      },
      isAddingProduct: false,
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
          this.products = this.sortedOnRating(products)
        })
        .catch(() => {
          setTimeout(this.getProducts, 5000)
        })
    },
    async addProduct (product) {
      await fetch('https://fakestoreapi.com/products', { method: 'POST', body: JSON.stringify(product) })
        .then(() => {
          this.getProducts()
        })
        .catch(() => {
          setTimeout(() => this.addProduct(product), 5000)
        })
    },
    async editProduct (product) {
      await fetch(`https://fakestoreapi.com/products/${product.id}`, { method: 'PATCH', body: JSON.stringify(product) })
        .then(() => {
          this.getProducts()
        })
        .catch(() => {
          setTimeout(() => this.editProduct(product), 5000)
        })
    },
    async deleteProduct (id) {
      await fetch(`https://fakestoreapi.com/products/${id}`, { method: 'DELETE' })
        .catch(() => this.deleteProduct(id), 5000)
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
    handleDeleteProduct (product, status) {
      this.products = this.products.filter((p) => p.id !== product.id)
      this.productsState[status] = this.productsState[status].filter((id) => id !== product.id)
      this.deleteProduct(product.id)
    },
    handleEditProduct (editedProduct) {
      this.editProduct(editedProduct)
    },
    handleChangeStatus (productId, prevStatus) {
      const nextStatus = (prevStatus === 'unprocessed') ? 'develop' : 'done'
      this.moveProduct(productId, prevStatus, nextStatus)
    },
    handleClickAddingForm () {
      this.isAddingProduct = !this.isAddingProduct
    },
    handleAddProduct () {
      this.isAddingProduct = false
      this.addProduct({ ...this.form })
      this.form = {
        description: '',
        price: null,
        title: '',
        category: '',
        image: null
      }
    },
    sortedOnRating (arr) { // сортируем каждый раз когда идет обновление
      return arr.sort((a, b) => (a.rating.rate < b.rating.rate) ? 1 : -1)
    },
    getColumnStyle (status) {
      switch (status) {
        case 'unprocessed':
          return { backgroundColor: 'Thistle' }
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
  min-width: 200px;
  padding: 20px;
  box-shadow: 0 4px 4px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  min-height: 800px;
  align-items: center;
}

.column h1 {
  margin: 0;
  text-align: center;
  color: #767c81;
}

.add-button {
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

.add-button:hover {
  background-color: blue;
}

.add-form {
  max-height: 100%;
  overflow-y: auto;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: justify;
  margin: 10px;
  color: #333;
  padding: 10px;
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
