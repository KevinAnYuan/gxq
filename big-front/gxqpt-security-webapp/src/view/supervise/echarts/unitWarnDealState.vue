<!-- 单位预警状态柱状图对比分析 -->
<template>
  <div id="unitWarnState"></div>
</template>

<script>
export default {
    data () {
        return {
            echartData: [],
            unitWarnState: '',
            monthData:[],
            handled:[],
            newCome:[],
            pending:[]
        }
    },
    methods: {
        initUnitWarnState () {
            // 基于准备好的dom，初始化echarts实例
            if (!this.unitWarnState) {
                this.unitWarnState = this.$echarts.init(document.getElementById('unitWarnState'))
            }
            const option = {
                backgroundColor: '#fff',
                tooltip: {
                    trigger: 'axis',
                    axisPointer: {
                        type: 'shadow'
                    }
                },
                legend: {
                    data: ['待处理', '产生', '已处理'],
                    align: 'right',
                    right: 10,
                    top: 20,
                    textStyle: {
                        color: "#000"
                    },
                    itemWidth: 10,
                    itemHeight: 10,
                    itemGap: 35
                },
                grid: {
                    left: '3%',
                    right: '4%',
                    bottom: '3%',
                    containLabel: true
                },
                xAxis: [{
                    type: 'category',
                    data: this.monthData.reverse(),
                    axisLine: {
                        show: true,
                        lineStyle: {
                            color: "#A6A6A6",
                            width: 1,
                            type: "solid"
                        }
                    },
                    axisTick: {
                        show: false,
                    },
                    axisLabel: {
                        show: true,
                        textStyle: {
                            color: "#00c7ff",
                        }
                    },
                }],
                yAxis: [{
                    type: 'value',
                    minInterval: 1,
                    axisLabel: {
                        formatter: '{value}'
                    },
                    axisTick: {
                        show: false,
                    },
                    axisLine: {
                        show: false,
                        lineStyle: {
                            color: "#00c7ff",
                            width: 1,
                            type: "solid"
                        },
                    },
                    splitLine: {
                        lineStyle: {
                            color: "#063374",
                        }
                    }
                }],
                series: [{
                    name: '待处理',
                    type: 'bar',
                    data: this.pending.reverse(),
                    barWidth: 10, //柱子宽度
                    barGap: 1, //柱子之间间距
                    itemStyle: {
                        normal: {
                            color: this.$echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                                offset: 0,
                                color: '#008cff'
                            }, {
                                offset: 1,
                                color: '#005193'
                            }]),
                            opacity: 1,
                        }
                    }
                }, {
                    name: '产生',
                    type: 'bar',
                    data: this.newCome.reverse(),
                    barWidth: 10,
                    barGap: 1,
                    itemStyle: {
                        color:"#f8d442"
                    }
                }, {
                    name: '已处理',
                    type: 'bar',
                    data: this.handled.reverse(),
                    barWidth: 10,
                    barGap: 1,
                    itemStyle: {
                        color: '#42f842',
                    }
                }]
            };
            // 使用刚指定的配置项和数据显示图表。
            this.unitWarnState.setOption(option);
        },
        refresh (data) {
            this.echartData = data;
            this.monthData = [];
            this.handled = [];
            this.newCome = [];
            this.pending = [];
            for(var i = 0;i<this.echartData.length;i++){
                this.monthData.push(this.echartData[i].month);
                this.handled.push(this.echartData[i].handled<=0 ? 0:this.echartData[i].handled);
                this.newCome.push(this.echartData[i].newCome<=0 ? 0:this.echartData[i].newCome);
                this.pending.push(this.echartData[i].pending<=0 ? 0:this.echartData[i].pending);
            }
            this.initUnitWarnState()
        }
    }
}
</script>

<style lang="less" scoped>
#unitWarnState{
  width: 100%;
  height: 300px;
}
</style>
