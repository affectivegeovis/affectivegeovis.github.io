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
            rowData: []
        }
    },
    watch: {
        selectedPointName(newName) {
            this.highlightPoint(newName);
            console.log("=====",newName)
        }
    },
    mounted() {
        this.loadData();
    },
    methods: {
        async loadData() {
            try {
                const response = await axios.get('/rc.json');
                const data = response.data;
                
                // 处理COL数据
                this.colData = data.col.map(item => ({
                    name: item.name,
                    x: parseFloat(item['0']),
                    y: parseFloat(item['1']),
                    z: parseFloat(item['2'])
                }));

                // 处理ROW数据
                this.rowData = data.row.map(item => ({
                    name: item.name,
                    x: parseFloat(item['0']),
                    y: parseFloat(item['1']),
                    z: parseFloat(item['2']),
                    cluster: item.cluster,
                    color: item.color
                }));

                this.initChart();
            } catch (error) {
                console.error('Error loading data:', error);
            }
        },

        highlightPoint(pointName) {
            if (!this.chart || !pointName) return;

            // const option = this.chart.getOption();
            // const series = option.series;
            console.log(pointName)
            // 重置所有点的大小
            // series.forEach(serie => {
            //     serie.symbolSize = serie.name === 'COL' ? 8 : 5;
            // });

            // // 找到并高亮选中的点
            // series.forEach(serie => {
            //     const data = serie.data;
            //     const index = data.findIndex(item => item[3] === pointName);
            //     if (index !== -1) {
            //         // 创建新的数据数组，只修改选中点的大小
            //         const newData = [...data];
            //         newData[index] = [...newData[index]];
            //         newData[index][4] = 15; // 设置选中点的大小
            //         serie.data = newData;
            //     }
            // });

            // this.chart.setOption({ series });
        },

        initChart() {
            this.chart = echarts.init(this.$refs.chartContainer);
            
            // 准备COL数据（三角形）
            const colSeries = {
                name: 'COL',
                type: 'scatter3D',
                data: this.colData.map(item => [
                    item.x,
                    item.y,
                    item.z,
                    item.name,
                    6 // 初始大小
                ]),
                symbol: 'triangle',
                symbolSize: 6,
                itemStyle: {
                    color: '#746f6e',
                    opacity: 0.3
                }
            };

            // 准备ROW数据（圆形，按cluster分组）
            const rowClusters = [...new Set(this.rowData.map(item => item.cluster))];
            const rowSeries = rowClusters.map(cluster => {
                const clusterData = this.rowData.filter(item => item.cluster === cluster);
                const color = clusterData[0].color;
                return {
                    name: `Cluster ${cluster}`,
                    type: 'scatter3D',
                    data: clusterData.map(item => [
                        item.x,
                        item.y,
                        item.z,
                        item.name,
                        5 // 初始大小
                    ]),
                    symbol: 'circle',
                    symbolSize: 5,
                    itemStyle: {
                        color: color,
                        opacity: 0.8
                    }
                };
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
                    data: ['COL', ...rowClusters.map(cluster => `Cluster ${cluster}`)]
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

            this.chart.setOption(option);

            // 添加点击事件
            this.chart.on('click', params => {
                if (params.data) {
                    this.$emit('point-clicked', params.data[3]);
                }
            });

            // 添加窗口大小变化事件
            window.addEventListener('resize', () => {
                this.chart.resize();
            });

            // 如果有选中的点，高亮显示
            if (this.selectedPointName) {
                this.highlightPoint(this.selectedPointName);
            }
        }
    },
    beforeDestroy() {
        if (this.chart) {
            this.chart.dispose();
        }
        window.removeEventListener('resize', this.chart.resize);
    }
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
    min-height: 300px;
    
}
</style>
  