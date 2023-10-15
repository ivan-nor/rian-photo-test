<template>
  <div class="card" v-if="product" :style="getCardStyle" :draggable="!isEditing">
    <img :src="product.image" alt="{{ product.title }}">
    <h4>{{ product.title }}</h4>
    <p class="category">{{ product.category }}</p>
    <p class="rating">Рейтинг: {{ product.rating.rate }} | Оценок: {{ product.rating.count }}</p>
    <p class="description" v-if="!isEditing">
      {{ product.description }}
    </p>
    <p class="price" v-if="!isEditing">{{ product.price }} руб.</p>
    <button class="edit-button" @click="editProduct" v-if="!isEditing">Редактировать</button>
    <button class="delete-button" @click="deleteProduct" v-if="!isEditing">Удалить</button>

    <div class="edit-form" v-if="isEditing" :style="getFormStyle">
      <h4>Редактировать товар</h4>
      <label for="editedDescription">Описание:</label>
      <textarea id="editedDescription" v-model="editedDescription" rows="10" autofocus></textarea>
      <label for="editedPrice">Цена:</label>
      <input type="number" id="editedPrice" v-model="editedPrice">
      <button @click="saveChanges">Сохранить</button>
      <button @click="editProduct">Отменить</button>
    </div>
    <button name="changeStatus" @click="changeStatus" v-if="status !== 'done'">{{ getButtonText }}</button>
  </div>
</template>

<script>

export default {
  name: 'CardComponent',
  props: {
    item: Object,
    status: String
  },
  data () {
    return {
      product: { ...this.item },
      isEditing: false,
      editedDescription: '',
      editedPrice: null
    }
  },
  created () {
    console.log(this.product)
  },
  methods: {
    editProduct () {
      this.isEditing = !this.isEditing
      this.editedDescription = this.product.description
      this.editedPrice = this.product.price
    },
    deleteProduct () {
      this.$emit('deleteProduct', this.product, this.status)
    },
    saveChanges () {
      this.isEditing = false
      this.product.description = this.editedDescription
      this.product.price = this.editedPrice
      this.$emit('editProduct', this.product)
    },
    changeStatus () {
      this.$emit('changeStatus', this.product.id, this.status)
    }
  },
  computed: {
    getCardStyle () {
      switch (this.status) {
        case 'unprocessed':
          return { backgroundColor: 'Lavender' }
        case 'develop':
          return { backgroundColor: 'LightYellow' }
        case 'done':
          return { backgroundColor: 'PaleGreen' }
        default:
          return { backgroundColor: 'white' }
      }
    },
    getFormStyle () {
      switch (this.status) {
        case 'unprocessed':
          return { backgroundColor: 'AliceBlue' }
        case 'develop':
          return { backgroundColor: 'Moccasin' }
        case 'done':
          return { backgroundColor: 'PaleGreen' }
        default:
          return { backgroundColor: 'gray' }
      }
    },
    getButtonText () {
      switch (this.status) {
        case 'unprocessed':
          return 'Взять в работу'
        case 'develop':
          return 'Завершить'
        case 'done':
          return null
        default:
          return null
      }
    }
  }
}
</script>

<style>
.card {
  border: 1px solid #e1d6d6;
  gap: 10px;
  padding: 15px;
  min-height: 100px;
  display: flex;
  flex-direction: column;
  align-items: justify;
  text-align: justify;
  box-shadow: 0 2px 7px #dfdfdf;
  border-radius: 10px;
  background-color: #fff;
  position: relative;
}

button {
  background-color: grey;
  color: #fff;
  border: none;
  padding: 5px;
  cursor: pointer;
  font-weight: bold;
  border-radius: 10px;
  font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
}

.delete-button:hover {
  background-color: red;
}

.edit-button:hover {
  background-color: yellowgreen;
}

button[name="changeStatus"] {
  background-color: rgb(11, 158, 11);
}

button[name="changeStatus"]:hover {
  background-color: rgb(56, 106, 23);
}

.edit-form {
  max-width: 100%;
  max-height: 100%;
  overflow-y: auto;
  padding: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  border-radius: 10px; /* Закругленные углы для формы */
  display: flex;
  flex-direction: column;
  align-items: justify;
  text-align: justify;
  margin: 5px 0;
  color: #333;
  gap: 5px;
}

p {
  margin: 0;
  font-size: 12px;
  line-height: 22px;
  color: #999;
}

.card:hover {
  box-shadow: 2px 2px 2px 2px rgba(255, 102, 51, 0.2);
}

.card h4 {
  margin: 0;
  font-weight: 500;
  display: block;
  text-transform: uppercase;
  color: #363636;
  text-decoration: none;
  transition: 0.3s;
}

.card h4:hover {
  color: #fbb72c;
}

.card img {
  max-width: 100%;
  max-height: 200px;
  width: auto;
  height: auto;
  object-fit: contain;
}

.price {
  font-weight: bold;
  font-size: 14px;
  color: #fbb72c;
  font-weight: 600;
  text-align: center;
}

.category, .rating {
  display: block;
  font-size: 10px;
  font-weight: 700;
  text-transform: uppercase;
  color: #fbb72c;
}
</style>
