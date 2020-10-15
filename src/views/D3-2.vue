<template>
  <div class="wrap">
    <h1>D3 - TPR</h1>
    <p>
      開始住院時間:
      <input v-model="startDate" type="date">
      <button style="margin-left:50px;" @click="init">
        random
      </button>
    </p>

    <div ref="content" class="content" />
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
      padding: 50,
      axisLineLg: 15,
      scaleX: null,
      scaleY: null,

      // 每天量測時間點
      timePeriods: ['07:30', '11:30', '14:00'],
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
      this.drawXChart()
      this.drawYChart()
      this.drawBtChart()
      this.drawPulseChart()
      this.drawRespChart()
    },
    drawXChart () {
      const chart = d3.select('#mySvg')
      this.scaleX = d3.scaleLinear().domain([0, 10]).range([0, this.width - this.padding * 2])

      // 顯示 X 坐標軸 - 日期
      const axisDateX = d3.axisTop(this.scaleX).tickFormat((d, i) => {
        const day = this.$moment(this.startDate).add(i, 'days')
        return day.month() + 1 + '/' + day.date()
      }).tickSizeInner(this.axisLineLg).tickSizeOuter((this.height - this.padding * 2) * -1)
      chart.append('g').call(axisDateX).attr('transform', `translate(${this.padding}, ${this.padding})`)
      // 顯示 X 坐標軸 - 時間
      const axisX = d3.axisTop(this.scaleX).ticks(40).tickFormat('')
      chart.append('g').call(axisX).attr('transform', `translate(${this.padding}, ${this.padding})`)
    },
    drawYChart () {
      const chart = d3.select('#mySvg')
      this.scaleY = d3.scaleLinear().domain([120, 0]).range([0, this.height - this.padding * 2])

      // 大刻度
      const axisLgBtY = d3.axisLeft(this.scaleY).ticks(20).tickFormat(d => d)
        .tickSizeInner(this.axisLineLg).tickSizeOuter((this.width - this.padding * 2) * -1)
      chart.append('g').call(axisLgBtY).attr('transform', `translate(50, ${this.padding})`)
      // 小刻度
      const axisSmBtY = d3.axisLeft(this.scaleY).ticks(100).tickFormat('')
      chart.append('g').call(axisSmBtY).attr('transform', `translate(50, ${this.padding})`)
    },
    drawBtChart () {
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
          return this.scaleY(item.bt)
        }).curve(d3.curveCardinal.tension(0.8))

      const group = d3.select('#mySvg').append('g').selectAll('g').data(this.datalist).enter().append('g')
      // 繪製座標點 - 體溫
      group
        .append('circle')
        .attr('cx', item => {
          const startDay = this.$moment(this.startDate)
          const day = this.$moment(item.date).diff(startDay, 'days')
          const hour = this.$moment(item.date).hour() * 60
          const min = this.$moment(item.date).minute()
          const aDayMin = 24 * 60
          return this.scaleX(day + (hour + min) / aDayMin)
        })
        .attr('cy', item => this.scaleY(item.bt))
        .attr('r', '3px')
        .attr('fill', 'blue')
        .attr('transform', `translate(${this.padding}, ${this.padding})`)
      // 繪製線段 - 體溫
      group
        .append('path')
        .attr('d', btline(this.datalist))
        .attr('stroke', 'blue')
        .attr('stroke-width', 0.3)
        .attr('fill', 'none')
        .attr('transform', `translate(${this.padding}, ${this.padding})`)
    },
    drawPulseChart () {
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
          return this.scaleY(item.pulse)
        }).curve(d3.curveCardinal.tension(0.8))

      const group = d3.select('#mySvg').append('g').selectAll('g').data(this.datalist).enter().append('g')
      // 繪製座標點 - 脈搏
      group
        .append('circle')
        .attr('cx', item => {
          const startDay = this.$moment(this.startDate)
          const day = this.$moment(item.date).diff(startDay, 'days')
          const hour = this.$moment(item.date).hour() * 60
          const min = this.$moment(item.date).minute()
          const aDayMin = 24 * 60
          return this.scaleX(day + (hour + min) / aDayMin)
        })
        .attr('cy', item => this.scaleY(item.pulse))
        .attr('r', '3px')
        .attr('fill', 'red')
        .attr('transform', `translate(${this.padding}, ${this.padding})`)
      // // 繪製線段 - 脈搏
      group
        .append('path')
        .attr('d', pulseline(this.datalist))
        .attr('stroke', 'red')
        .attr('stroke-width', 0.3)
        .attr('fill', 'none')
        .attr('transform', `translate(${this.padding}, ${this.padding})`)
    },
    drawRespChart () {
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
          return this.scaleY(item.resp)
        }).curve(d3.curveCardinal.tension(0.8))

      const group = d3.select('#mySvg').append('g').selectAll('g').data(this.datalist).enter().append('g')
      // 繪製座標點 - 脈搏
      group
        .append('circle')
        .attr('cx', item => {
          const startDay = this.$moment(this.startDate)
          const day = this.$moment(item.date).diff(startDay, 'days')
          const hour = this.$moment(item.date).hour() * 60
          const min = this.$moment(item.date).minute()
          const aDayMin = 24 * 60
          return this.scaleX(day + (hour + min) / aDayMin)
        })
        .attr('cy', item => this.scaleY(item.resp))
        .attr('r', '3px')
        .attr('fill', 'black')
        .attr('transform', `translate(${this.padding}, ${this.padding})`)
      // 繪製線段 - 脈搏
      group
        .append('path')
        .attr('d', respline(this.datalist))
        .attr('stroke', 'black')
        .attr('stroke-width', 0.3)
        .attr('fill', 'none')
        .attr('transform', `translate(${this.padding}, ${this.padding})`)
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
  }
  .content {
    flex-grow: 1;
  }
  .gridline line{
    stroke: #ccc;
  }
  .axisbt line{
    stroke: blue;
    stroke-width:0.5;
  }
  .axisbt text{
    stroke: blue;
    stroke-width:0.5;
  }
  .axispulse line{
    stroke: red;
    stroke-width:0.5;
  }
  .axispulse text{
    stroke: red;
    stroke-width:0.5;
  }
  .axisresp line{
    stroke: black;
    stroke-width:0.5;
  }
  .axisresp text{
    stroke: black;
    stroke-width:0.5;
  }
</style>
