<template>
  <div class="card" v-if="product">
    <h6>{{ product.title }}</h6>
    <p class="category">{{ product.category }}</p>
    <!-- <img :src="product.image" alt="{{ product.title }}"> -->
    <!-- <p class="description" v-if="!isEditing">{{ product.description }}</p> -->
    <p class="price" v-if="!isEditing">{{ product.price }} руб.</p>
    <p class="rating">Рейтинг: {{ product.rating }}</p>

    <button class="edit-button" @click="editCard" v-if="!isEditing">Редактировать</button>
    <div class="edit-form" v-if="isEditing">
      <h3>Редактировать товар</h3>
      <label for="editedDescription">Описание:</label>
      <textarea id="editedDescription" v-model="editedDescription"></textarea>
      <label for="editedPrice">Цена:</label>
      <input type="number" id="editedPrice" v-model="editedPrice">
      <button @click="saveChanges">Сохранить</button>
      <button @click="cancelEdit">Отменить</button>
    </div>
    <button @click="setPrevStatus">Set Prev Status</button>
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
  }
}
</script>

<style>
.card {
  border: 3px solid #161515;
  padding: 10px;
  height: auto;
  display: flex;
  flex-direction: column;
  align-items: justify;
  text-align: justify;
}

.edit-button {
  background-color: #ad3288;
  color: #fff;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
}

.card p {
  margin: 0; /* Убираем отступы внутри параграфа */
}

h2 {
  margin: 10px 0;
}

img {
  max-width: 100%;
  height: auto;
}

.card img {
  max-width: 100%;
  max-height: 100%;
  width: auto; /* Добавляем ширину auto, чтобы изображение могло изменять размер */
  height: auto; /* Добавляем высоту auto, чтобы изображение могло изменять размер */
  object-fit: contain; /* Масштабировать изображение так, чтобы оно полностью помещалось */
}

.description {
  margin: 10px 0;
}

.price {
  font-weight: bold;
}

.category {
  color: #888;
}

.rating {
  color: #f39c12;
}
</style>
