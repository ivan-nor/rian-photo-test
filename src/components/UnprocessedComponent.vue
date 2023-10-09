<template>
  <div></div>
</template>

<script>
export default {
  name: 'TasksComponent',
  props: {
    unprocessed: Array,
    products: Array
  },
  data () {
    return {
    }
  },
  created () {
    console.log('CREAATE Unprocessed')
  }
}
</script>

<style scoped lang="scss">

</style>
