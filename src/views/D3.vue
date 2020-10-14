<template>
  <div>
    <h1>D3 - TPR</h1>
  </div>
</template>

<script>
import * as d3 from 'd3'

export default {
  name: 'D3',
  data () {
    return {
      startDate: '2020/10/01',
      endDate: '2020/10/20',
      datalist: [
        { date: '2020/10/01 09:30' }
      ]
    }
  },
  mounted () {
    console.log(d3)
  }
}
</script>
