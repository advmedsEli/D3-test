<template>
  <div>
    <h1>test</h1>
    <svg class="chart" />
  </div>
</template>

<script>
import * as d3 from 'd3'

export default {
  name: 'Test',
  mounted () {
  // 以下小弟打靶成績，中心點為 (50, 50)
    const shot = [
      { x: 10, y: 96 },
      { x: 90, y: 13 },
      { x: 77, y: 89 },
      { x: 25, y: 35 },
      { x: 13, y: 6 }
    ]

    // 設定 svg 的寬高
    const chart = d3
      .select('.chart')
      .attr('width', 500)
      .attr('height', 500)

    // 選取所有的 g 群組
    const group = chart
      .selectAll('g')
      .data(shot)
      .enter()
      .append('g')

    // 取 X 軸比例尺
    const scaleX = d3
      .scaleLinear()
      .domain([0, 100])
      .range([0, 300])

    // 顯示 X 坐標軸
    const axisX = d3.axisBottom(scaleX)
    chart
      .append('g')
      .call(axisX)
      .attr('transform', 'translate(30,330)')

    // 取 Y 軸比例尺
    const scaleY = d3
      .scaleLinear()
      .domain([100, 0])
      .range([0, 300])

    // 顯示 Y 坐標軸
    const axisY = d3.axisLeft(scaleY)
    chart
      .append('g')
      .call(axisY)
      .attr('transform', 'translate(30,30)')

    // 繪製座標點
    group
      .append('circle')
      .attr('cx', item => scaleX(item.x))
      .attr('cy', item => scaleY(item.y))
      .attr('r', '2px')
      .attr('height', 25 - 2)
      .attr('fill', 'red')
      .attr('transform', 'translate(30,30)')

    // 顯示座標
    // group
    //   .append('text')
    //   .attr('x', item => scaleX(item.x))
    //   .attr('y', item => scaleY(item.y))
    //   .style('fill', 'black')
    //   .style('font-size', '10px')
    //   .style('font-weight', 'bold')
    //   .text(item => `(${item.x},${item.y})`)
    //   .attr('transform', 'translate(35,25)')
  }
}
</script>
