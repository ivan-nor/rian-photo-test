<template>
  <div></div>
</template>

<script>
export default {
  name: 'TasksComponent',
  props: {
    done: Array,
    products: Array
  },
  data () {
    return {
    }
  },
  created () {
    console.log('CREATE DONE')
  }
}
</script>

<style scoped lang="scss">

</style>
