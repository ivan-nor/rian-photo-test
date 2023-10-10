<template>
  <div class="column" :style="calculateColumnStyle">
    <CardComponent :status="status" :style="calculateCardStyle" v-for="product of tasks" :key="product.id"/>
  </div>
</template>

<script>
import CardComponent from './CardComponent.vue'

export default {
  name: 'ColumnComponent',
  components: { CardComponent },
  props: {
    idiesOfTasks: Array,
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
        unprocessed: (t) => (t === 'card') ? 'lightblue' : 'blue',
        develop: (t) => (t === 'card') ? 'limon' : 'yellow',
        done: (t) => (t === 'card') ? 'cyan' : 'green'
      }

      return { backgroundColor: mappedStyles[this.status](type) }
    }
  },
  created () {
    console.log('CREAATE Column', this.status, this.idiesOfTasks)
  },
  computed: {
    tasks () {
      console.log('COLUMN computed tasks', this.idiesOfTasks, this.products, this.status)
      return this.products.filter((product) => this.idiesOfTasks.includes(product.id))
    },
    calculateColumnStyle () {
      return this.getComputedStyle(this.status, 'column')
    },
    calculateCardStyle () {
      return this.getComputedStyle(this.status, 'card')
    }
  },
  updated () {
    // console.log('COLUMN tasks', this.tasks)
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
