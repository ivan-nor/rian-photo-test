<template>
  <div class="column" :style="calculateColumnStyle">
    <CardComponent
    :status="status"
    :style="calculateCardStyle"
    v-for="item of tasks"
    :key="item.id"
    :item="item"
    @saveChanges="handleEditCardInternal"
    @setStatus="handleSetStatusCardInternal"
  />
  </div>
</template>

<script>
import CardComponent from './CardComponent.vue'

export default {
  name: 'ColumnComponent',
  components: { CardComponent },
  props: {
    idiesOfProducts: Array,
    products: Array,
    status: String
  },
  data () {
    return {
    }
  },
  methods: {
    getComputedStyle (status, type) {
      const mappedStyles = {
        unprocessed: (t) => (t === 'card') ? 'DeepSkyBlue' : 'LightSteelBlue',
        develop: (t) => (t === 'card') ? 'Moccasin' : 'khaki',
        done: (t) => (t === 'card') ? 'MediumSpringGreen' : 'lightgreen'
      }

      return { backgroundColor: mappedStyles[this.status](type) }
    },
    handleEditCardInternal (cardId, editedDescription, editedPrice) {
      // Обрабатываем событие save cahnge из CardComponent в ColumnComponent
      // Затем передаем его вверх до App
      this.$emit('saveChanges', cardId, editedDescription, editedPrice)
    },
    handleSetStatusCardInternal (cardId, prevStatus) {
      this.$emit('setStatus', cardId, prevStatus)
    }
  },
  computed: {
    tasks () {
      console.log('COLUMN computed tasks', this.idiesOfProducts, this.products, this.status)
      return this.products.filter((product) => this.idiesOfProducts.includes(product.id))
    },
    calculateColumnStyle () {
      return this.getComputedStyle(this.status, 'column')
    },
    calculateCardStyle () {
      return this.getComputedStyle(this.status, 'card')
    }
  },
  created () {
    // console.log('COLUMN created')
  }
}
</script>

<style>
.column {
  display: flex;
  flex-direction: column;
  gap: 10px; /* Расстояние между карточками внутри колонки */
  max-width: 300px; /* Максимальная ширина колонки */
  background-color: #3498db;
  padding: 10px;
}
</style>
