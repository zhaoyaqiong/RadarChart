<template>
    <div class="container">
        <svg width="100%" height="100%"></svg>
    </div>
</template>

<script>
    import * as d3 from "d3";

    export default {
        name: "RadarChart",
        data() {
            return {
                chartData: {
                    fieldNames: ['语文', '数学', '外语', '物理', '化学', '生物', '政治', '历史', '地理'],
                    values: [
                        [10, 20, 30, 40, 50, 60, 70, 80, 90],
                        [90, 80, 70, 60, 50, 40, 30, 20, 10]
                    ]
                },
                chartConfig: {
                    radius: 100,
                    // 指标的数量，和fieldNames的长度相同
                    total: 9,
                    // 网轴的级数，网轴上从小到大有多少个正方形
                    level: 4,
                    rangeMin: 0,
                    rangeMax: 100,
                    arc: 2 * Math.PI
                }
            }
        },
        methods: {
            getColor(idx) {
                let palette = [
                    '#2ec7c9', '#b6a2de', '#5ab1ef', '#ffb980', '#d87a80',
                    '#8d98b3', '#e5cf0d', '#97b552', '#95706d', '#dc69aa',
                    '#07a2a4', '#9a7fd1', '#588dd5', '#f5994e', '#c05050',
                    '#59678c', '#c9ab00', '#7eb00a', '#6f5553', '#c14089'
                ];
                return palette[idx % palette.length];
            },
            createRadarChart() {
                let that = this;
                let height = 300, width = 600;
                let main = d3.select('.container svg')
                    .append('g')
                    .classed('main', true)
                    .attr('transform', "translate(" + width / 2 + ',' + height / 2 + ')');
                let onePiece = this.chartConfig.arc / this.chartConfig.total;
                let polygons = {
                    webs: [],
                    webPoints: []
                };
                for (let k = this.chartConfig.level; k > 0; k--) {
                    let webs = '', webPoints = [];
                    let r = this.chartConfig.radius / this.chartConfig.level * k;
                    for (let i = 0; i < this.chartConfig.total; i++) {
                        let x = r * Math.sin(i * onePiece),
                            y = r * Math.cos(i * onePiece);
                        webs += x + ',' + y + ' ';
                        webPoints.push({
                            x: x,
                            y: y
                        })
                    }
                    polygons.webs.push(webs);
                    polygons.webPoints.push(webPoints);
                }
                let webs = main.append('g')
                    .classed('webs', true);
                webs.selectAll('polygon')
                    .data(polygons.webs)
                    .enter()
                    .append('polygon')
                    .attr('points', function (d) {
                        return d;
                    });
                let lines = main.append('g')
                    .classed('lines', true);
                lines.selectAll('line')
                    .data(polygons.webPoints[0])
                    .enter()
                    .append('line')
                    .attr('x1', 0)
                    .attr('y1', 0)
                    .attr('x2', function (d) {
                        return d.x
                    })
                    .attr('y2', function (d) {
                        return d.y;
                    });
                let areasData = this.calcAreaData();
                // 添加g分组包含所有雷达图区域
                let areas = main.append('g')
                    .classed('areas', true);
                // 添加g分组包含雷达图区域下的多边形和圆点
                areas.selectAll('g')
                    .data(areasData)
                    .enter()
                    .append('g')
                    .attr('class', function (d, i) {
                        return 'area' + (i + 1);
                    });
                for (let i = 0; i < areasData.length; i++) {
                    // 遍历每个雷达图区域
                    let area = areas.select('.area' + (i + 1)),
                        areaData = areasData[i];
                    area.append('polygon')
                        .attr('points', areaData.polygon)
                        .attr('stroke', function (d, index) {
                            return that.getColor(i);
                        })
                        .attr('fill', function (d, index) {
                            return that.getColor(i)
                        });
                    // 绘制雷达图定点
                    let circles = area.append('g')
                        .classed('circles', true);
                    circles.selectAll('circles')
                        .data(areaData.points)
                        .enter()
                        .append('circle')
                        .attr('cx', function (d) {
                            return d.x
                        })
                        .attr('cy', function (d) {
                            return d.y
                        })
                        .attr('r', 3)
                        .attr('stroke', function (d, index) {
                            return that.getColor(i);
                        });
                }
                // 计算文字坐标
                let textPoints = [];
                let textRadius = this.chartConfig.radius + 20;
                for (let i = 0; i < this.chartConfig.total; i++) {
                    let x = textRadius * Math.sin(i * onePiece);
                    let y = textRadius * Math.cos(i * onePiece);
                    textPoints.push({
                        x: x,
                        y: y
                    })
                }
                // 添加到画布
                let texts = main.append('g')
                    .classed('texts', true);
                texts.selectAll('text')
                    .data(textPoints)
                    .enter()
                    .append('text')
                    .attr('x', function (d) {
                        return d.x
                    })
                    .attr('y', function (d) {
                        return d.y
                    })
                    .text(function (d, i) {
                        return that.chartData.fieldNames[i];
                    })
            },
            calcAreaData(){
                let areasData = [];
                let onePiece = this.chartConfig.arc / this.chartConfig.total;
                let values = this.chartData.values;
                for (let i = 0; i < values.length; i++) {
                    let value = values[i],
                        area = '',
                        points = [];
                    for (let k = 0; k < this.chartConfig.total; k++) {
                        let r = this.chartConfig.radius * (value[k] - this.chartConfig.rangeMin) / (this.chartConfig.rangeMax - this.chartConfig.rangeMin);
                        let x = r * Math.sin(k * onePiece),
                            y = r * Math.cos(k * onePiece);
                        area += x + ',' + y + ' ';
                        points.push({
                            x: x,
                            y: y
                        })
                    }
                    areasData.push({
                        polygon: area,
                        points: points
                    })
                }
                return areasData;
            },
            updateRadarChart(fieldName, number, value){
                let index = this.chartData.fieldNames.indexOf(fieldName);
                if (index === -1){
                    return null;
                }
                this.chartData.values[number][index] = value;
                let areasData = this.calcAreaData();
                // 添加g分组包含所有雷达图区域
                let main = d3.select('.container svg .main');
                let areas = main.select('.areas');
                for (let i = 0; i < areasData.length; i++) {
                    // 遍历每个雷达图区域
                    let area = areas.select('.area' + (i + 1)),
                        areaData = areasData[i];
                    area.selectAll('polygon')
                        .transition()
                        .duration(5000)
                        .attr('points', areaData.polygon);
                    // 绘制雷达图定点
                    let circles = area.selectAll('.circles');
                    circles.selectAll('circle')
                        .data(areaData.points)
                        .transition()
                        .duration(5000)
                        .attr('cx', function (d) {
                            return d.x
                        })
                        .attr('cy', function (d) {
                            return d.y
                        });
                }
            }
        },
        mounted() {
            this.createRadarChart();
            this.updateRadarChart('地理',1, 100);
        }
    }
</script>

<style>
    .container {
        margin: 30px auto;
        width: 600px;
        height: 300px;
        border: 1px solid #000;
    }

    .webs polygon,
    .lines line {
        fill: white;
        fill-opacity: 0.5;
        stroke: gray;
        stroke-dasharray: 10, 5;
    }

    .areas polygon {
        fill-opacity: 0.5;
        stroke-width: 3;
    }

    .areas circle {
        fill: white;
        stroke-width: 3;
    }

    .texts text {
        font-size: 14px;
        text-anchor: middle;
    }

    .webs polygon:nth-child(odd) {
        fill: lightgray;
    }
</style>
