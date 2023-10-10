<template>
  <div class="card">
    <h2>{{ product.title }}</h2>
    <p class="category">{{ product.category }}</p>
    <img :src="product.image" alt="{{ product.title }}">
    <p class="description" v-if="!isEditing">{{ product.description }}</p>
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
  </div>
</template>

<script>

export default {
  name: 'CardComponent',
  props: {
    product: Object,
    status: String
  },
  data () {
    return {}
  }
}
</script>

<style>
.card {
  background-color: #89a6b1;
  border: 1px solid #ddd;
  padding: 10px;
  height: auto;
  display: flex;
  flex-direction: column;
  align-items: center;
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
