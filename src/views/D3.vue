<template>
  <div class="wrap">
    <h1>D3 - TPR</h1>
    <p>開始住院時間: <input v-model="startDate" type="date" @change="getFakeData"></p>
    <div class="content">
      <svg id="chart" />
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3'

export default {
  name: 'D3',
  data () {
    return {
      // 每天量測時間點
      timePeriods: ['09:30', '11:30', '14:00'],
      startDate: '2020-10-01',
      // date:測量時間, bt:體溫, pulse:脈搏, resp:呼吸, nbpsys:收縮壓, nbpdis:舒張壓
      datalist: []
    }
  },
  mounted () {
    // this.init()
    // this.getFakeData()
    // d3.select('.content').append('svg')
    //   .attr('width', 960)
    //   .attr('height', 500)
    //   .call(this.responsivefy)
    this.init()
  },
  methods: {
    // responsivefy (svg) {
    // // get container + svg aspect ratio
    //   var container = d3.select(svg.node().parentNode)
    //   var width = parseInt(svg.style('width'))
    //   var height = parseInt(svg.style('height'))
    //   var aspect = width / height

    //   // add viewBox and preserveAspectRatio properties,
    //   // and call resize so that svg resizes on inital page load
    //   svg.attr('viewBox', '0 0 ' + width + ' ' + height)
    //     .attr('perserveAspectRatio', 'xMinYMid')
    //     .call(resize)

    //   // to register multiple listeners for same event type,
    //   // you need to add namespace, i.e., 'click.foo'
    //   // necessary if you call invoke this function for multiple svgs
    //   // api docs: https://github.com/mbostock/d3/wiki/Selections#on
    //   d3.select(window).on('resize.' + container.attr('id'), resize)

    //   // get width of container and resize svg to fit it
    //   function resize () {
    //     var targetWidth = parseInt(container.style('width'))
    //     svg.attr('width', targetWidth)
    //     svg.attr('height', Math.round(targetWidth / aspect))
    //   }
    // },
    init () { // 前公司每個月產品的產量
      const report = [
        { month: 1, quantity: 8561 },
        { month: 2, quantity: 9347 },
        { month: 3, quantity: 10335 },
        { month: 4, quantity: 9900 },
        { month: 5, quantity: 12012 },
        { month: 6, quantity: 10300 },
        { month: 7, quantity: 13020 },
        { month: 8, quantity: 15300 },
        { month: 9, quantity: 17210 },
        { month: 10, quantity: 13400 },
        { month: 11, quantity: 11059 },
        { month: 12, quantity: 9900 }
      ]

      // 設定 svg 的寬高
      const chart = d3
        .select('#chart')
        .attr('width', 500)
        .attr('height', 500)

      // 選取所有的 g 群組
      const group = chart
        .selectAll('g')
        .data(report)
        .enter()
        .append('g')

      // 取 X 軸比例尺
      const scaleX = d3
        .scaleLinear()
        .domain([0, d3.max(report.map(item => item.month))])
        .range([0, 300])

      // 顯示 X 坐標軸
      const axisX = d3.axisBottom(scaleX)
      chart
        .append('g')
        .call(axisX)
        .attr('transform', 'translate(50,330)')

      // 取 Y 軸比例尺
      const scaleY = d3
        .scaleLinear()
        .domain([d3.max(report.map(item => item.quantity)), 0])
        .range([0, 300])

      // 顯示 Y 坐標軸
      const axisY = d3.axisLeft(scaleY)
      chart
        .append('g')
        .call(axisY)
        .attr('transform', 'translate(50,30)')

      // 定義線段
      const line = d3
        .line()
        .x(item => scaleX(item.month))
        .y(item => scaleY(item.quantity))
        .curve(d3.curveBasis)

      // 將線段繪製出來
      group
        .append('path')
        .attr('d', line(report))
        .attr('transform', 'translate(50,30)')
        .attr('stroke', 'black')
        .attr('stroke-width', 1)
        .attr('fill', 'none')
    },
    getFakeData () {
      const fakeData = []
      for (let i = 0; i < 10; i++) {
        const day = this.$moment(this.startDate).add(i, 'days').format('L')
        this.timePeriods.forEach(timePeriod => {
          fakeData.push({
            date: `${day} ${timePeriod}`,
            bt: this.getBt(),
            pulse: this.getPulse(),
            resp: this.getResp(),
            nbpsys: this.getNbpsys(),
            nbpdis: this.getNbpdis()
          })
        })
      }
      this.datalist = fakeData
    },
    getBt () {
      // 體溫: 35 ~ 37.5
      return 35 + Number((Math.random() * 2.6).toFixed(1))
    },
    getPulse () {
      // 脈搏: 60 ~ 90
      return 60 + Math.floor(Math.random() * 31)
    },
    getResp () {
      // 呼吸: 12 ~ 18
      return 12 + Math.floor(Math.random() * 7)
    },
    getNbpsys () {
      // 收縮壓: 120 ～ 139
      return 120 + Math.floor(Math.random() * 20)
    },
    getNbpdis () {
      // 舒張壓: 80 ~ 89
      return 80 + Math.floor(Math.random() * 10)
    }

  }
}
</script>

<style>
  .wrap {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
  }
  .content {
    flex-grow: 1;
  }
  /* .chart {
    width: 100%;
    height: 100%;
  } */
</style>
