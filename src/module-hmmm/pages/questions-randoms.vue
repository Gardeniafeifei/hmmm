<template>
  <div class="dashboard-container">
    <div class="app-container">{{dataList}}</div>
  </div>
</template>

<script>
import { simple } from '@/api/hmmm/questions'
export default {
  name: 'QuestionsRandoms',
  data() {
    return {
      dataList: {}
    }
  },
  mounted() {
    this.initData()
  },
  methods: {
    async initData() {
      const { data: res } = await simple({})
      this.dataList = res 
    }
  }
}
</script>

<style scoped>
</style>
