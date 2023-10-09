<template>
  <div></div>
</template>

<script>
export default {
  name: 'TasksComponent',
  props: {
    develop: Array,
    products: Array
  },
  data () {
    return {
    }
  },
  created () {
    console.log('CREATED Develop')
  }
}
</script>

<style scoped lang="scss">

</style>
