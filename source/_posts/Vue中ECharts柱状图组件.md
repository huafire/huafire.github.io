---
title: Vue中ECharts柱状图组件
tags:
  - Vue
  - ECharts
categories:
  - Vue
  - ECharts
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: fc48d151
date: 2024-04-19 16:37:14
---

柱状图效果如下：

<img src="E:/Hexo/source/_posts/Vue中ECharts柱状图组件/柱状图效果.png" style="zoom: 80%;" />

```vue
<template>
    <div style="width: 100%; height: 400px" />
</template>

<script>
import echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import resize from '../dashboard/mixins/resize'

export default {
    mixins: [resize],
    // 因为柱状图参数较多，直接在props中返回，便于调用
    props: {
        data: {
            type: Object,
            default: () => {
                return {
                    title: "",
                    series: [
                        { name: "1", type: "bar", data: [1, 2] },
                        { name: "2", type: "bar", data: [2, 3] },
                    ],
                    xAxisData: []
                }
            }
        }
        
    },
    data() {
        return {
            chart: null
        }
    },
    mounted() {
        this.$nextTick(() => {
            this.initChart()
        })
    },
    beforeDestroy() {
        if (!this.chart) {
            return
        }
        this.chart.dispose()
        this.chart = null
    },
    watch: {
        // 如果数据发生变化，自动刷新柱状图
        data: {
            handler: function () {
                this.initChart()
            },
            // 遍历到所有参数
            deep: true
        },
    },
    methods: {
        initChart() {
            this.chart = echarts.init(this.$el, 'macarons')
            const legendData = []
            this.data.series.forEach(data => {
                legendData.push(data.name || '没有名字')
            })
            this.chart.setOption({
                // 标题
                title: {
                    text: this.data.title,
                },
                tooltip: {
                    trigger: 'axis'
                },
                legend: {
                    data: legendData
                },
                toolbox: {
                    show: true,
                    feature: {
                        dataView: { show: true, readOnly: false },
                        magicType: { show: true, type: ['line', 'bar'] },
                        restore: { show: true },
                        saveAsImage: { show: true }
                    }
                },
                calculable: true,
                xAxis: [
                    {
                        type: 'category',
                        data: this.data.xAxisData,
                        // data: ['2024-04-15', '2024-04-16', '2024-04-17', '2024-04-18', '2024-04-19', '2024-04-20', '2024-04-21']
                    }
                ],
                yAxis: [
                    // 如果是两组数据，则放两个
                    {
                        type: 'value'
                    },
                ],
                series: this.data.series
            })
        }
    }
}
</script>
```

​	补充：`watch`中的方法，只要数据改变，则刷新柱状图

​	在另个一组件中调用时，方法如下：

```vue
<template>
	<div>
		<Techart class="echart" :data="echartData" style="height: 492px" />
	</div>
</template>

<script>
export default {
    data() {
        return {
            // 柱状图参数
            echartData: {
                title: "新增授权",
                series: [
                    { name: "新增授权", type: "bar", data: [] },
                    { name: "数据访问", type: "bar", data: [] },
                ],
                xAxisData: []
            }
        }
    },
	// 使标签生效
    components: {
        Techart,
    },
}
</script>
```

​	调用完成。

​	补充：`mounted`中的方法，在渲染前就会执行。
