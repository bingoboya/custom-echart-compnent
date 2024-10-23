<template>
    <button @click="handleClick">loading</button>
    <button @click="handleEdit">修改图1 X轴</button>
    <div style="padding: 20px;">
        <textarea ref="el" class="resizer" v-text="text" />
    </div>
    <div>
        {{ size.width }} - {{ size.height }}
    </div>
    <div class="container_wrapper">
        <div class="chart_container1">
            <div ref="chartRefWrap" class="chart_item" v-for="(item, index) in options" :key="index">

                <Echart width="100%" height="305px" :loading="loading" :option="item as ECOption"
                    :on-mouseover="onMouseover" :on-mouseout="onMouseout" />
                <!-- <Echart width="100%" height="100%" :loading="loading" :option="item as ECOption" :on-mouseover="onMouseover" :on-mouseout="onMouseout" /> -->
            </div>
        </div>
    </div>
    <div v-if="false" class="chart-container">11
        <!-- <div class="chart_item" v-for="(item, index) in options" :key="index"> -->
        <!-- <Echart
                width="100%"
                height="305px"
                :loading="loading"
                :option="item as ECOption"
                :on-mouseover="onMouseover"
                :on-mouseout="onMouseout"
            /> -->
        <!-- </div> -->
    </div>

</template>
<script setup lang="ts">
import { onMounted, ref, reactive } from 'vue';
// import { vElementSize } from '@vueuse/components';
import { useElementSize } from '@vueuse/core'
import Echart from '@/components/BaseEchart/index.vue';
import type { ECOption } from '@/components/BaseEchart/config';
import type { EChartsType } from 'echarts/core';
import type { YAXisOption } from 'echarts/types/dist/shared';

const options: any = ref<ECOption[]>([]);
const options1: any = ref<ECOption[]>([]);
const loading = ref(false);
const chartRefWrap = ref(null);

// const size = reactive(
//   useElementSize(
//     chartRefWrap.value
//   ),
// )

const el = ref(null)
const size = reactive(
    useElementSize(
        el,
        { width: 0, height: 0 },
        // { box: 'border-box' },
    ),
)

const text = ref(size)

// 通过给 ECharts title 注册鼠标 over 与 out 事件，来控制 yAxis.name 的 color，从而达到 “口径” 的显示与隐藏功能
const onMouseover = (_: object, chart: EChartsType, option: ECOption) => {
    (option.yAxis as YAXisOption).nameTextStyle.color = '#000';
    chart.setOption(option);
};

const onMouseout = (_: object, chart: EChartsType, option: ECOption) => {
    (option.yAxis as YAXisOption).nameTextStyle.color = 'transparent';
    chart.setOption(option);
};

const handleClick = () => {
    loading.value = !loading.value;
};
const handleEdit = () => {
    options.value[0].xAxis.data = [
        '06:00',
        '07:00',
        '08:00',
        '09:00',
        '10:00',
        '11:00',
    ];
};

const fetch = () => {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(
                Array.from({ length: 3 }, (_, index) => ({
                    title: {
                        text: '用鼠标 hover 我' + index,
                        textStyle: {
                            rich: {
                                title: {
                                    width: 12,
                                    height: 12,
                                },
                            },
                        },
                        triggerEvent: true,
                    },
                    legend: { right: '4%' },
                    tooltip: { trigger: 'axis' },
                    grid: {
                        left: '3%',
                        right: '4%',
                        bottom: '3%',
                        containLabel: true,
                    },
                    xAxis: {
                        boundaryGap: false,
                        overflow: 'truncate',
                        data: [
                            '00:00',
                            '01:00',
                            '02:00',
                            '03:00',
                            '04:00',
                            '05:00',
                        ],
                    },
                    yAxis: {
                        name: '我会隐藏和显现',
                        nameTextStyle: {
                            color: 'transparent',
                            padding: [0, 0, 0, 50],
                        },
                        type: 'value',
                        scale: true,
                        splitLine: {
                            lineStyle: { color: '#283593', type: 'dashed' },
                        },
                    },
                    series: [
                        {
                            name: '当前',
                            data: ['31', '41', '31', '21', '11', '07'],
                            type: 'line',
                            symbol: 'circle',
                            showSymbol: false,
                            sampling: 'lttb',
                            smooth: 0.2,
                        },
                        {
                            name: '昨日',
                            data: ['12', '31', '41', '11', '21', '11'],
                            type: 'line',
                            symbol: 'circle',
                            showSymbol: false,
                            sampling: 'lttb',
                            smooth: 0.2,
                        },
                        {
                            name: '上周',
                            data: ['24', '11', '31', '01', '53', '61'],
                            type: 'line',
                            symbol: 'circle',
                            showSymbol: false,
                            sampling: 'lttb',
                            smooth: 0.2,
                        },
                    ],
                    color: ['#007BEE', '#00FFF4', '#EEC67A'],
                }))
            );
        }, 1000);
    });
};


const fetch1 = () => {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(
                Array.from({ length: 1 }, (_, index) => ({
                    title: {
                        text: '用鼠标 hover 我' + index,
                        textStyle: {
                            rich: {
                                title: {
                                    width: 12,
                                    height: 12,
                                },
                            },
                        },
                        triggerEvent: true,
                    },
                    legend: { right: '4%' },
                    tooltip: { trigger: 'axis' },
                    grid: {
                        left: '3%',
                        right: '4%',
                        bottom: '3%',
                        containLabel: true,
                    },
                    xAxis: {
                        boundaryGap: false,
                        overflow: 'truncate',
                        data: [
                            '00:00',
                            '01:00',
                            '02:00',
                            '03:00',
                            '04:00',
                            '05:00',
                        ],
                    },
                    yAxis: {
                        name: '我会隐藏和显现',
                        nameTextStyle: {
                            color: 'transparent',
                            padding: [0, 0, 0, 50],
                        },
                        type: 'value',
                        scale: true,
                        splitLine: {
                            lineStyle: { color: '#283593', type: 'dashed' },
                        },
                    },
                    series: [
                        {
                            name: '当前',
                            data: ['31', '41', '31', '21', '11', '07'],
                            type: 'line',
                            symbol: 'circle',
                            showSymbol: false,
                            sampling: 'lttb',
                            smooth: 0.2,
                        },
                        {
                            name: '昨日',
                            data: ['12', '31', '41', '11', '21', '11'],
                            type: 'line',
                            symbol: 'circle',
                            showSymbol: false,
                            sampling: 'lttb',
                            smooth: 0.2,
                        },
                        {
                            name: '上周',
                            data: ['24', '11', '31', '01', '53', '61'],
                            type: 'line',
                            symbol: 'circle',
                            showSymbol: false,
                            sampling: 'lttb',
                            smooth: 0.2,
                        },
                    ],
                    color: ['#007BEE', '#00FFF4', '#EEC67A'],
                }))
            );
        }, 1000);
    });
};

onMounted(async () => {
    const data = await fetch();
    options.value = data as ECOption[];
    const data1 = await fetch1();
    options1.value = data1 as ECOption[];


    //     const { width, height } = useElementSize(
    //     chartRefWrap.value
    //   )
    //     // const { width, height } = useElementSize(chartRefWrap)

    //     // console.log(1111, size);
    //     console.log(222, width, height);
});
</script>



<style scoped>
.resizer {
    resize: both;
    padding: 1rem;
    width: 200px;
    height: 200px;
    /* border: 1px solid var(--vp-c-divider); */
    border-radius: 4px;
    background: #fff;
    outline: none;
    white-space: pre;
    overflow-wrap: normal;
    overflow: hidden;
}
.container_wrapper {
    border: 1px solid;
    width: 80%;
    height: 80%;
    overflow: hidden;
    background: #ffc0cb66;
}

.chart_container1 {
    display: flex;
    gap: 30px;
    flex-direction: column;
    width: 100%;
    height: 100%;
    /* min-width: 600px; */
    margin: 0 auto;
    background-color: #f9f9f9;
    padding: 10px;
}

.chart_item {
    /* min-width: 600px; */
    background: rgba(89, 196, 89, 0.276);
    border: 2px solid rgb(248, 43, 2);
}

.chart-container {
    display: grid;
    grid-row-gap: 15px;
    grid-template-rows: 1fr 1fr 1fr;
    width: 50%;
    min-width: 600px;
    margin: 0 auto;
    background-color: #f9f9f9;
}
</style>
