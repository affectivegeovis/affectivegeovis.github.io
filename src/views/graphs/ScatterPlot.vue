<template>
    <div class="scatter-container">
        <div ref="chartContainer" class="chart-area"></div>
    </div>
</template>

<script>
import axios from 'axios';
import * as echarts from 'echarts';
import 'echarts-gl';

export default {
    name: 'ScatterPlot',
    props: {
        selectedPointName: {
            type: String,
            default: ''
        }
    },
    data() {
        return {
            chart: null,
            colData: [],
            rowData: [],
            colormap:[
                "#F3D027",
                "#AAD2E0",
                "#478F8D",
                "#23BF0C",//3
            ]
        }
    },
    watch: {
        selectedPointName(newName) {
            this.highlightPoint(newName);
            // console.log("=====",newName)
        }
    },
    mounted() {
        this.initChart();
    },
    methods: {
        highlightPoint(pointName) {
            
            if (!this.chart || !pointName) return;
            // console.log(pointName)
            let option = this.chart.getOption();
            let series1 = option.series;
            series1.forEach(serie => {
                const defaultSize = serie.name === 'Method' ? 8 : 5; // 如果是 "Method"，默认大小为 8，否则为 5
                serie.data.forEach(item => {
                    item[4] = defaultSize; // 设置默认大小
                });
                const index = serie.data.findIndex(item => item[3] === pointName);
                if (index !== -1) {
                    serie.data[index][4] = 15; // 设置选中点的大小
                }
                
                    
            });
            console.log(option)
            this.chart.setOption(option)
        },
        initChart() {
            const echarts = require('echarts');
            this.chart = echarts.init(this.$refs.chartContainer);
            const myChart = this.chart;
            let _this=this
            axios.get('/rc.json').then((response) => {
                const data = response.data;
                
                // 处理COL数据
                _this.colData = data.col.map(item => ({
                    name: item.name,
                    x: parseFloat(item['0']),
                    y: parseFloat(item['1']),
                    z: parseFloat(item['2'])
                }));

                // 处理ROW数据
                _this.rowData = data.row.map(item => ({
                    name: item.name,
                    x: parseFloat(item['0']),
                    y: parseFloat(item['1']),
                    z: parseFloat(item['2']),
                    cluster: item.cluster,
                    color: item.color
                }));
            // 准备COL数据（三角形）
            const colSeries = {
                name: 'Method',
                type: 'scatter3D',
                data: _this.colData.map(item => [
                    item.x,
                    item.y,
                    item.z,
                    item.name,
                    6
                    
                ]),
                symbol: 'triangle',
                symbolSize: 6,
                itemStyle: {
                    color: '#746f6e',
                    opacity: 0.3
                }
            };
            const clustername={
                0: 'Computational',
                1: 'Anthropological',
                2: 'Art',
                3: 'Social activism',
            }
            // 准备ROW数据（圆形，按cluster分组）
            const rowClusters = [...new Set(_this.rowData.map(item => item.cluster))];
            const rowSeries = rowClusters.map(cluster => {
                const clusterData = _this.rowData.filter(item => item.cluster === cluster);
                const color = clusterData[0].color;
                return {
                    name: `${clustername[cluster]}`,
                    type: 'scatter3D',
                    data: clusterData.map(item => [
                        item.x,
                        item.y,
                        item.z,
                        item.name,
                        5 ,
                    ]),
                    symbol: 'circle',
                    symbolSize: function (val) {
                        console.log(val[4])
                        return val[4] || 5; // 第5个元素控制大小，默认5
                    },
                    itemStyle: {
                        color:function (val) {
                            if (val.value[4]===15){
                                return 'red'
                            }
                            return color; // 第5个元素控制大小，默认5
                    },
                        opacity: 0.8
                    }
                }
         
            });

            const option = {
                tooltip: {
                    formatter: (params) => {
                        return `Name：${params.data[3]}<br/>
                                X: ${params.data[0].toFixed(2)}<br/>
                                Y: ${params.data[1].toFixed(2)}<br/>
                                Z: ${params.data[2].toFixed(2)}`;
                    }
                },
                legend: {
                    data: ['Method', ...rowClusters.map(cluster => `${clustername[cluster]}`)]
                },
                grid3D: {
                    viewControl: {
                        projection: 'orthographic',
                        autoRotate: false
                    },
                    boxHeight: 80
                },
                xAxis3D: {
                    type: 'value',
                    name: 'X'
                },
                yAxis3D: {
                    type: 'value',
                    name: 'Y'
                },
                zAxis3D: {
                    type: 'value',
                    name: 'Z'
                },
                series: [colSeries, ...rowSeries]
            };

            myChart.setOption(option);

            // 添加点击事件
            myChart.on('click', params => {
                if (params.data) {
                    this.$emit('point-clicked', params.data[3]);
                }
            });

            // 添加窗口大小变化事件
            window.addEventListener('resize', () => {
                myChart.resize();
            });

            // 如果有选中的点，高亮显示
            // if (this.selectedPointName) {
            //     this.highlightPoint(this.selectedPointName);
            // }
        
    }).catch((error) => {
  console.error('Error reading local JSON file:', error);
}); 
      }
            
    },

    // beforeDestroy() {
    //     if (this.chart) {
    //         this.chart.dispose();
    //     }
    //     window.removeEventListener('resize', this.chart.resize);
    // }
}
</script>

<style scoped>
.scatter-container {
    width: 100%;
    height: 100%;
    min-width: 300px;
    position: relative;
}

.chart-area {
    width: 100%;
    height: 100%;
    min-height: 250px;
    
}
</style>
  