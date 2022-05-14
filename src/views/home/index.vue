<!-- home -->
<template>
  <div class="list-container">
    <div class="item" v-for="(item,index) in list" :key="index">{{item}}</div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      list: []
    }
  },

  computed: {},
  mounted() {
    for (let i = 0; i < 60; i++) {
      this.list.push(i)
    }
  },

  methods: {}
}
</script>
<style lang="scss" scoped>
.list-container {
  background: #fff;
  padding: 12px;
  display: flex;
  justify-content: flex-start;
  flex-wrap: wrap;
  .item {
    width: 80px;
    height: 80px;
    border: 1px solid green;
  }
}
</style>
