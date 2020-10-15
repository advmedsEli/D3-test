<template>
  <div class="wrap">
    <h1>D3 - TPR</h1>
    <p>開始住院時間: <input v-model="startDate" type="date"></p>
    <button @click="init">
      random
    </button>
    <div ref="content" class="content" />
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
    init () {
      this.$refs.content.innerHTML = ''
      this.getFakeData()
      // 設定 svg 的寬高
      const chart = d3
        .select('.content')
        .append('svg')
        .attr('width', 1300)
        .attr('height', 780)

      // 選取所有的 g 群組
      const group = chart
        .selectAll('g')
        .data(this.datalist)
        .enter()
        .append('g')

      // 取 X 軸比例尺
      const scaleX = d3.scaleLinear().domain([0, 10]).range([0, 1100])
      // Grid - 顯示 X 坐標軸
      const gridX = d3.axisTop(scaleX).tickFormat('').tickSizeInner(-700)
      chart.append('g').call(gridX).attr('class', 'gridline').attr('transform', 'translate(100,50)')
      // 取 Y 軸比例尺
      const scaleY = d3.scaleLinear().domain([50, 30]).range([0, 700])
      // Grid - 顯示 Y 坐標軸
      const gridY = d3.axisLeft(scaleY).ticks(100).tickFormat('').tickSizeInner(-1100)
      chart.append('g').call(gridY).attr('class', 'gridline').attr('transform', 'translate(100,50)')

      // 顯示 X 坐標軸
      const axisX = d3.axisTop(scaleX).tickFormat((d, i) => {
        const day = this.$moment(this.startDate).add(i, 'days')
        return day.month() + 1 + '/' + day.date()
      }).tickSizeInner(15).tickSizeOuter(-700)
      chart.append('g').call(axisX).attr('transform', 'translate(100,50)')

      // 顯示 Y 坐標軸
      const axisY = d3.axisLeft(scaleY).ticks(100).tickFormat(d => {
        if (d % 0.5 === 0) return d
        else return ''
      }).tickSizeInner(15).tickSizeOuter(-1100)
      chart.append('g').call(axisY).attr('transform', 'translate(100,50)')

      // 定義線段
      const btline = d3.line()
        .x((item, i) => {
          const startDay = this.$moment(this.startDate)
          const day = this.$moment(item.date).diff(startDay, 'days')
          const hour = this.$moment(item.date).hour() * 24
          const min = this.$moment(item.date).minute()
          const aDayMin = 24 * 60
          return scaleX(day + (hour + min) / aDayMin)
        })
        .y((item, i) => {
          return scaleY(item.bt)
        })

      // 將線段繪製出來
      group
        .append('path')
        .attr('d', btline(this.datalist))
        .attr('transform', 'translate(100,50)')
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
  .gridline line{
    stroke: #ccc;
  }
</style>
