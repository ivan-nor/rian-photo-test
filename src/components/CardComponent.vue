<template>
  <div class="card" v-if="product" :style="getCardStyle">
    <h4>{{ product.title }}</h4>
    <p class="category">{{ product.category }}</p>
    <p class="description" v-if="!isEditing">
      <img :src="product.image" alt="{{ product.title }}">
      {{ product.description }}
    </p>
    <p class="price" v-if="!isEditing">{{ product.price }} руб.</p>
    <p class="rating">Рейтинг: {{ product.rating }}</p>
    <button class="edit-button" @click="editCard" v-if="!isEditing">Редактировать</button>

    <div class="edit-form" v-if="isEditing" :style="getFormStyle">
      <h3>Редактировать товар</h3>
      <label for="editedDescription">Описание:</label>
      <textarea id="editedDescription" v-model="editedDescription" rows="10" autofocus></textarea>
      <label for="editedPrice">Цена:</label>
      <input type="number" id="editedPrice" v-model="editedPrice">
      <button @click="saveChanges">Сохранить</button>
      <button @click="cancelEdit">Отменить</button>
    </div>
    <button name="set-prev-status" @click="setPrevStatus" v-if="status !== 'done'">{{ getButtonText }}</button>
    <!-- <button @click="setNextStatus">Set Next Status</button> -->
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
  methods: {
    editCard () {
      // Открываем форму редактирования
      console.log('click editCard')
      this.isEditing = true
      // Заполняем поля формы текущими значениями описания и цены
      this.editedDescription = this.product.description
      this.editedPrice = this.product.price
    },
    saveChanges () {
      // Сохраняем измененные значения
      console.log('click saveChanges')
      this.product.description = this.editedDescription
      this.product.price = parseFloat(this.editedPrice)
      // Закрываем форму редактирования
      this.isEditing = false
      // Генерируем событие и передаем данные в родительский компонент (App.vue)
      this.$emit('saveChanges', this.product.id, this.editedDescription, this.editedPrice)
    },
    cancelEdit () {
      console.log('click cancelEdit')
      // Отменяем редактирование и закрываем форму
      this.isEditing = false
    },
    setPrevStatus () {
      this.$emit('setStatus', this.product.id, this.status)
    }
  },
  computed: {
    getCardStyle () {
      return { backgroundColor: 'white' }
      // switch (this.status) {
      //   case 'unprocessed':
      //     return { backgroundColor: 'Plum' }
      //   case 'develop':
      //     return { backgroundColor: 'Khaki' }
      //   case 'done':
      //     return { backgroundColor: 'LightGreen' }
      //   default:
      //     return { backgroundColor: 'gray' }
      // }
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
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  background-color: #fff;
}

.edit-button,
button[name="set-prev-status"] {
  background-color: #ad3288;
  color: #fff;
  border: none;
  padding: 5px;
  cursor: pointer;
  font-weight: bold;
  font-family: Arial, sans-serif;
  border-radius: 10px;
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
  font-family: Arial, sans-serif;
}

.card p {
  margin: 0;
  font-size: x-small;
}

.card h4 {
  margin: 0;
}

.card img {
  max-width: 100px;
  max-height: 100px;
  width: auto;
  height: auto;
  object-fit: contain;
  float: left;
  margin-right: 6px;
}

.description {
  font-family: Arial, sans-serif;
}

.price {
  font-weight: bold;
  font-family: Arial, sans-serif;
}

.category {
  font-family: Arial, sans-serif;
}

.rating {
  font-family: Arial, sans-serif;
}
</style>
