<template>
  <div class="chart_wrapper">
    <div>
      <button @click="handleClick">loading</button>
      <button @click="handleEdit">修改图1 X轴</button>
    </div>
    <div class="chart_container">
      <div ref="chartRefWrap" class="chart_item" style="height: 100%; width: 100%;">
        <Echart width="100%" height="100%" :loading="loading" :option="newOptions" :on-mouseover="onMouseover"
          :on-mouseout="onMouseout" />
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
import { onMounted, ref, toRaw, watch } from 'vue';
import Echart from '@/components/BaseEchart/index.vue';
import type { ECOption } from '@/components/BaseEchart/config';
import type { EChartsType } from 'echarts/core';
import type { YAXisOption } from 'echarts/types/dist/shared';

const props = defineProps({
  option: {
    type: Object,
    default: () => {
      return {
        tooltip: {
          trigger: 'item'
        },
        legend: {
          top: '5%',
          left: 'center'
        },
        series: [
          {
            name: 'Access From',
            type: 'pie',
            radius: ['40%', '70%'],
            avoidLabelOverlap: false,
            itemStyle: {
              borderRadius: 10,
              borderColor: '#fff',
              borderWidth: 2
            },
            label: {
              show: false,
              position: 'center'
            },
            emphasis: {
              label: {
                show: true,
                fontSize: 40,
                fontWeight: 'bold'
              }
            },
            labelLine: {
              show: false
            },
            data: [
              { value: 1048, name: 'Search Engine' },
              { value: 735, name: 'Direct' },
              { value: 580, name: 'Email' },
              { value: 484, name: 'Union Ads' },
              { value: 300, name: 'Video Ads' }
            ]
          }
        ]
      }
    }
  }
})

const newOptions: any = ref<ECOption[]>([]);
const loading = ref(false);
const chartRefWrap = ref(null)


watch(() => toRaw(props.option), (newValue, oldValue) => {
  newOptions.value = newValue as ECOption[];
}, { immediate: true })

onMounted(async () => {
  // const data2 = await fetch();
  // newOptions.value = data2 as ECOption[];
});


// 通过给 ECharts title 注册鼠标 over 与 out 事件，来控制 yAxis.name 的 color，从而达到 “口径” 的显示与隐藏功能
const onMouseover = (_: object, chart: EChartsType, option: ECOption) => {
  // (option.yAxis as YAXisOption).nameTextStyle.color = '#000';
  chart.setOption(option);
};

const onMouseout = (_: object, chart: EChartsType, option: ECOption) => {
  // (option.yAxis as YAXisOption).nameTextStyle.color = 'transparent';
  chart.setOption(option);
};

const handleClick = () => {
  loading.value = !loading.value;
};
const handleEdit = () => {
  newOptions.value.xAxis.data = [
    '06:00',
    '07:00',
    '08:00',
    '09:00',
    '10:00',
    '11:00',
  ];
};
</script>



<style scoped>
.chart_wrapper {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.chart_container {
  display: flex;
  gap: 30px;
  width: 100%;
  height: 100%;
  margin: 0 auto;
  background-color: #f9f9f9;
  padding: 10px;
  justify-items: center;
  align-items: center;
  align-content: center;
  justify-content: center;

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
