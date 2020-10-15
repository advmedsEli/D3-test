<template>
  <div class="wrap">
    <h1>D3 - TPR</h1>
    <div style="display:flex;align-items:center;">
      <p>開始住院時間: <input v-model="startDate" type="date"></p>
      <button style="margin-left:50px;" @click="init">
        random
      </button>
      <div class="mark">
        <p class="mark-child mark-bt">
          <span />體溫
        </p>
        <p class="mark-child mark-pulse">
          <span />脈搏
        </p>
        <p class="mark-child mark-resp">
          <span />呼吸
        </p>
        <p class="mark-child mark-nbp">
          收縮壓 / 舒張壓
        </p>
      </div>
    </div>
    <div ref="content" class="content" />
    <table>
      <thead>
        <tr>
          <th>日期</th>
          <th>體溫</th>
          <th>脈搏</th>
          <th>呼吸</th>
          <th>收縮壓</th>
          <th>舒張壓</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, index) in datalist" :key="index">
          <td v-for="(item, i) in row" :key="i">
            {{ item }}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import * as d3 from 'd3'

export default {
  name: 'D3',
  data () {
    return {
      width: 1200,
      height: 750,
      padding: 30,
      paddingLeft: 150,
      axisLineLg: 15,
      scaleX: null,

      // 每天量測時間點
      timePeriods: ['07:30', '11:30', '16:00'],
      startDate: '2020-10-01',
      // date:測量時間, bt:體溫, pulse:脈搏, resp:呼吸, nbpsys:收縮壓, nbpdis:舒張壓
      datalist: []
    }
  },
  mounted () {
    this.init()
  },
  methods: {
    responsivefy (svg) {
    // get container + svg aspect ratio
      var container = d3.select(svg.node().parentNode)
      var width = parseInt(svg.style('width'))
      var height = parseInt(svg.style('height'))
      var aspect = width / height

      // add viewBox and preserveAspectRatio properties,
      // and call resize so that svg resizes on inital page load
      svg.attr('viewBox', '0 0 ' + width + ' ' + height)
        .attr('perserveAspectRatio', 'xMinYMid')
        .call(resize)

      // to register multiple listeners for same event type,
      // you need to add namespace, i.e., 'click.foo'
      // necessary if you call invoke this function for multiple svgs
      // api docs: https://github.com/mbostock/d3/wiki/Selections#on
      d3.select(window).on('resize.' + container.attr('id'), resize)

      // get width of container and resize svg to fit it
      function resize () {
        var targetWidth = parseInt(container.style('width'))
        svg.attr('width', targetWidth)
        svg.attr('height', Math.round(targetWidth / aspect))
      }
    },
    init () {
      this.$refs.content.innerHTML = ''
      this.getFakeData()

      d3.select('.content').append('svg').attr('id', 'mySvg').attr('width', this.width).attr('height', this.height)
      this.drawGrid()

      this.drawDateAxis()
      this.drawBtChart()
      this.drawPulseChart()
      this.drawRespChart()
      this.drawNbpChart()
    },
    drawGrid () {
      // X 軸
      this.scaleX = d3.scaleLinear().domain([0, 10]).range([0, this.width - this.paddingLeft - this.padding])
      const axisX = d3.axisTop(this.scaleX).ticks(20).tickSizeInner((this.height - this.padding * 2) * -1).tickFormat('')
      d3.select('#mySvg').append('g').call(axisX).attr('class', 'gridline').attr('transform', `translate(${this.paddingLeft}, ${this.padding})`)
      // Y 軸
      const scaleY = d3.scaleLinear().domain([160, 40]).range([0, this.height - this.padding * 2])
      const axisSmBtY = d3.axisLeft(scaleY).ticks(65).tickSizeInner((this.width - this.paddingLeft - this.padding) * -1).tickFormat('')
      d3.select('#mySvg').append('g').call(axisSmBtY).attr('class', 'gridline').attr('transform', `translate(${this.paddingLeft}, ${this.padding})`)
    },
    drawDateAxis () {
      // 顯示 X 坐標軸 - 日期
      const axisDateX = d3.axisTop(this.scaleX).tickFormat((d, i) => {
        const day = this.$moment(this.startDate).add(i, 'days')
        return day.month() + 1 + '/' + day.date()
      }).tickSizeInner(this.axisLineLg).tickSizeOuter((this.height - this.padding * 2) * -1)
      d3.select('#mySvg').append('g').call(axisDateX).attr('transform', `translate(${this.paddingLeft}, ${this.padding})`)
      // 顯示 X 坐標軸 - 時間
      // const axisX = d3.axisTop(this.scaleX).ticks(20).tickFormat('')
      // d3.select('#mySvg').append('g').call(axisX).attr('transform', `translate(${this.paddingLeft}, ${this.padding})`)
    },
    drawBtChart () {
      const chart = d3.select('#mySvg')
      const scaleY = d3.scaleLinear().domain([41, 35]).range([0, this.height - this.padding * 2])

      // 正常範圍 35 ~ 37.5, 圖表範圍 35 ~ 41
      // 大刻度
      const axisLgBtY = d3.axisLeft(scaleY).ticks(13).tickFormat(d => d)
        .tickSizeInner(this.axisLineLg)
      chart.append('g').call(axisLgBtY).attr('class', 'axisbt').attr('transform', `translate(50, ${this.padding})`)
      // 小刻度
      // const axisSmBtY = d3.axisLeft(scaleY).ticks(65).tickFormat('')
      // chart.append('g').call(axisSmBtY).attr('class', 'axisbt').attr('transform', `translate(50, ${this.padding})`)

      // 定義線段
      const btline = d3.line()
        .x(item => {
          const startDay = this.$moment(this.startDate)
          const day = this.$moment(item.date).diff(startDay, 'days')
          const hour = this.$moment(item.date).hour() * 60
          const min = this.$moment(item.date).minute()
          const aDayMin = 24 * 60
          return this.scaleX(day + (hour + min) / aDayMin)
        })
        .y((item, i) => {
          return scaleY(item.bt)
        })

      const group = chart.append('g').selectAll('g').data(this.datalist).enter().append('g')
      // 繪製座標點 - 體溫
      // group
      //   .append('circle')
      //   .attr('cx', item => {
      //     const startDay = this.$moment(this.startDate)
      //     const day = this.$moment(item.date).diff(startDay, 'days')
      //     const hour = this.$moment(item.date).hour() * 60
      //     const min = this.$moment(item.date).minute()
      //     const aDayMin = 24 * 60
      //     return this.scaleX(day + (hour + min) / aDayMin)
      //   })
      //   .attr('cy', item => scaleY(item.bt))
      //   .attr('r', '3px')
      //   .attr('fill', '#0c79c2')
      //   .attr('transform', `translate(${this.paddingLeft}, ${this.padding})`)
      // 繪製線段 - 體溫
      group
        .append('path')
        .attr('d', btline(this.datalist))
        .attr('stroke', '#0c79c2')
        .attr('stroke-width', '2px')
        .attr('fill', 'none')
        .attr('transform', `translate(${this.paddingLeft}, ${this.padding})`)
    },
    drawPulseChart () {
      const chart = d3.select('#mySvg')
      const scaleY = d3.scaleLinear().domain([160, 40]).range([0, this.height - this.padding * 2])

      // 正常範圍 60 ~ 90, 圖表範圍 40 ~ 160
      // 大刻度
      const axisLgBtY = d3.axisLeft(scaleY).ticks(13).tickFormat(d => d)
        .tickSizeInner(this.axisLineLg)
      chart.append('g').call(axisLgBtY).attr('class', 'axispulse').attr('transform', `translate(100, ${this.padding})`)
      // 小刻度
      // const axisSmBtY = d3.axisLeft(scaleY).ticks(65).tickFormat('')
      // chart.append('g').call(axisSmBtY).attr('class', 'axispulse').attr('transform', `translate(100, ${this.padding})`)

      // 定義線段
      const pulseline = d3.line()
        .x(item => {
          const startDay = this.$moment(this.startDate)
          const day = this.$moment(item.date).diff(startDay, 'days')
          const hour = this.$moment(item.date).hour() * 60
          const min = this.$moment(item.date).minute()
          const aDayMin = 24 * 60
          return this.scaleX(day + (hour + min) / aDayMin)
        })
        .y((item, i) => {
          return scaleY(item.pulse)
        })

      const group = chart.append('g').selectAll('g').data(this.datalist).enter().append('g')
      // 繪製座標點 - 脈搏
      // group
      //   .append('circle')
      //   .attr('cx', item => {
      //     const startDay = this.$moment(this.startDate)
      //     const day = this.$moment(item.date).diff(startDay, 'days')
      //     const hour = this.$moment(item.date).hour() * 60
      //     const min = this.$moment(item.date).minute()
      //     const aDayMin = 24 * 60
      //     return this.scaleX(day + (hour + min) / aDayMin)
      //   })
      //   .attr('cy', item => scaleY(item.pulse))
      //   .attr('r', '3px')
      //   .attr('fill', '#28a745')
      //   .attr('transform', `translate(${this.paddingLeft}, ${this.padding})`)
      // 繪製線段 - 脈搏
      group
        .append('path')
        .attr('d', pulseline(this.datalist))
        .attr('stroke', '#28a745')
        .attr('stroke-width', '2px')
        .attr('fill', 'none')
        .attr('transform', `translate(${this.paddingLeft}, ${this.padding})`)
    },
    drawRespChart () {
      const chart = d3.select('#mySvg')
      const scaleY = d3.scaleLinear().domain([70, 10]).range([0, this.height - this.padding * 2])

      // 正常範圍 12 ~ 18, 圖表範圍 10 ~ 70
      // 大刻度
      const axisLgBtY = d3.axisLeft(scaleY).ticks(13).tickFormat(d => d)
        .tickSizeInner(this.axisLineLg).tickSizeOuter((this.width - this.paddingLeft - this.padding) * -1)
      chart.append('g').call(axisLgBtY).attr('class', 'axisresp').attr('transform', `translate(${this.paddingLeft}, ${this.padding})`)
      // 小刻度
      // const axisSmBtY = d3.axisLeft(scaleY).ticks(65).tickFormat('')
      // chart.append('g').call(axisSmBtY).attr('class', 'axisresp').attr('transform', `translate(${this.paddingLeft}, ${this.padding})`)

      // 定義線段
      const respline = d3.line()
        .x(item => {
          const startDay = this.$moment(this.startDate)
          const day = this.$moment(item.date).diff(startDay, 'days')
          const hour = this.$moment(item.date).hour() * 60
          const min = this.$moment(item.date).minute()
          const aDayMin = 24 * 60
          return this.scaleX(day + (hour + min) / aDayMin)
        })
        .y((item, i) => {
          return scaleY(item.resp)
        })

      const group = chart.append('g').selectAll('g').data(this.datalist).enter().append('g')
      // 繪製座標點 - 脈搏
      // group
      //   .append('circle')
      //   .attr('cx', item => {
      //     const startDay = this.$moment(this.startDate)
      //     const day = this.$moment(item.date).diff(startDay, 'days')
      //     const hour = this.$moment(item.date).hour() * 60
      //     const min = this.$moment(item.date).minute()
      //     const aDayMin = 24 * 60
      //     return this.scaleX(day + (hour + min) / aDayMin)
      //   })
      //   .attr('cy', item => scaleY(item.resp))
      //   .attr('r', '3px')
      //   .attr('fill', '#000')
      //   .attr('transform', `translate(${this.paddingLeft}, ${this.padding})`)
      // 繪製線段 - 脈搏
      group
        .append('path')
        .attr('d', respline(this.datalist))
        .attr('stroke', '#000')
        .attr('stroke-width', '2px')
        .attr('fill', 'none')
        .attr('transform', `translate(${this.paddingLeft}, ${this.padding})`)
    },
    drawNbpChart () {
      const scaleY = d3.scaleLinear().domain([160, 40]).range([0, this.height - this.padding * 2])
      const group = d3.select('#mySvg').append('g').selectAll('g').data(this.datalist).enter().append('g')
      // 繪製座標點 - 收縮壓
      group
        .append('polygon')
        .attr('points', item => {
          const startDay = this.$moment(this.startDate)
          const day = this.$moment(item.date).diff(startDay, 'days')
          const hour = this.$moment(item.date).hour() * 60
          const min = this.$moment(item.date).minute()
          const aDayMin = 24 * 60
          const centerX = this.scaleX(day + (hour + min) / aDayMin)
          const centerY = scaleY(item.nbpsys)
          return `${centerX - 5} ${centerY - 5}, ${centerX} ${centerY + 5}, ${centerX + 5} ${centerY - 5}`
        })
        .attr('fill', '#dc3545')
        .attr('transform', `translate(${this.paddingLeft}, ${this.padding})`)
      // 繪製座標點 - 舒張壓
      group
        .append('polygon')
        .attr('points', item => {
          const startDay = this.$moment(this.startDate)
          const day = this.$moment(item.date).diff(startDay, 'days')
          const hour = this.$moment(item.date).hour() * 60
          const min = this.$moment(item.date).minute()
          const aDayMin = 24 * 60
          const centerX = this.scaleX(day + (hour + min) / aDayMin)
          const centerY = scaleY(item.nbpdis)
          return `${centerX - 5} ${centerY + 5}, ${centerX} ${centerY - 5}, ${centerX + 5} ${centerY + 5}`
        })
        .attr('fill', '#dc3545')
        .attr('transform', `translate(${this.paddingLeft}, ${this.padding})`)
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
      return 35.5 + Number((Math.random() * 1.1).toFixed(1))
    },
    getPulse () {
      // 脈搏: 60 ~ 90
      return 65 + Math.floor(Math.random() * 16)
    },
    getResp () {
      // 呼吸: 12 ~ 18
      return 12 + Math.floor(Math.random() * 7)
    },
    getNbpsys () {
      // 收縮壓: 120 ～ 139
      return 125 + Math.floor(Math.random() * 10)
    },
    getNbpdis () {
      // 舒張壓: 80 ~ 89
      return 82 + Math.floor(Math.random() * 5)
    }
  }
}
</script>

<style>
  .wrap {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
    margin: 0 10px;
  }
  .content {
    flex-grow: 1;
  }
  .gridline line {
    stroke: #ccc;
  }
  .axisbt line, .axisbt path{
    stroke: #0c79c2;
    stroke-width: 1;
  }
  .axisbt text{
    color: #0c79c2;
  }
  .axispulse line, .axispulse path{
    stroke: #28a745;
    stroke-width: 1;
  }
  .axispulse text{
    color: #28a745;
  }
  .axisresp line, .axisresp path{
    stroke: #000;
    stroke-width: 1;
  }
  .axisresp text{
    color: #000;
  }
  table {
    border-collapse: collapse;
    border: 1px solid black;
    margin: 0 20px 50px;
  }
  table, th, td{
    border: 1px solid black;
  }
  .mark {
    display: flex;
    margin-left: 50px;
  }
  .mark p:not(:last-child) {
    margin-right: 20px;
  }
  .mark-child span {
    display: inline-block;
    width: 50px;
    height: 7px;
    background-color: black;
    border-radius: 2px;
    margin-right: 5px;
  }
  .mark-bt span {
    background-color: #0c79c2;
  }
  .mark-pulse span {
    background-color: #28a745;
  }
  .mark-resp span {
    background-color: #000;
  }
  .mark-nbp::before {
    content: '';
    display: inline-block;
    width: 0;
    height: 0;
    border: 7px solid transparent;
    border-bottom: 7px solid #dc3545;
    margin-right: -5px;
  }
</style>
