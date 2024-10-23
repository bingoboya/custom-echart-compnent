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

const newOptions: any = ref<any[]>([]);
const loading = ref(false);
const chartRefWrap = ref(null)

// 生成扇形的曲面参数方程，用于 series-surface.parametricEquation
const getParametricEquation = (
  startRatio: any,
  endRatio: any,
  isSelected: any,
  isHovered: any,
  k: any,
  h: any
) => {
  // 计算
  let midRatio = (startRatio + endRatio) / 2;

  let startRadian = startRatio * Math.PI * 2;
  let endRadian = endRatio * Math.PI * 2;
  let midRadian = midRatio * Math.PI * 2;

  // 如果只有一个扇形，则不实现选中效果。
  // if (startRatio === 0 && endRatio === 1) {
  //     isSelected = false;
  // }
  isSelected = false;
  // 通过扇形内径/外径的值，换算出辅助参数 k（默认值 1/3）
  k = typeof k !== "undefined" ? k : 1 / 3;

  // 计算选中效果分别在 x 轴、y 轴方向上的位移（未选中，则位移均为 0）
  let offsetX = isSelected ? Math.sin(midRadian) * 0.1 : 0;
  let offsetY = isSelected ? Math.cos(midRadian) * 0.1 : 0;

  // 计算高亮效果的放大比例（未高亮，则比例为 1）
  let hoverRate = isHovered ? 1.05 : 1;

  // 返回曲面参数方程
  return {
    u: {
      min: -Math.PI,
      max: Math.PI * 3,
      step: Math.PI / 32,
    },

    v: {
      min: 0,
      max: Math.PI * 2,
      step: Math.PI / 20,
    },

    x: function (u: any, v: any) {
      if (u < startRadian) {
        return (
          offsetX +
          Math.cos(startRadian) * (1 + Math.cos(v) * k) * hoverRate
        );
      }
      if (u > endRadian) {
        return (
          offsetX +
          Math.cos(endRadian) * (1 + Math.cos(v) * k) * hoverRate
        );
      }
      return offsetX + Math.cos(u) * (1 + Math.cos(v) * k) * hoverRate;
    },

    y: function (u: any, v: any) {
      if (u < startRadian) {
        return (
          offsetY +
          Math.sin(startRadian) * (1 + Math.cos(v) * k) * hoverRate
        );
      }
      if (u > endRadian) {
        return (
          offsetY +
          Math.sin(endRadian) * (1 + Math.cos(v) * k) * hoverRate
        );
      }
      return offsetY + Math.sin(u) * (1 + Math.cos(v) * k) * hoverRate;
    },

    z: function (u: any, v: any) {
      if (u < -Math.PI * 0.5) {
        return Math.sin(u);
      }
      if (u > Math.PI * 2.5) {
        return Math.sin(u) * h * 0.1;
      }
      return Math.sin(v) > 0 ? 1 * h * 0.1 : -1;
    },
  };
}

// 生成模拟 3D 饼图的配置项
const getPie3D = (pieData: any, internalDiameterRatio: number) => {
  let series = [];
  let sumValue = 0;
  let startValue = 0;
  let endValue = 0;
  let legendData = [];
  let k =
    typeof internalDiameterRatio !== "undefined"
      ? (1 - internalDiameterRatio) / (1 + internalDiameterRatio)
      : 1 / 3;

  // 为每一个饼图数据，生成一个 series-surface 配置
  for (let i = 0; i < pieData.length; i++) {
    sumValue += pieData[i].value;

    let seriesItem: any = {
      name:
        typeof pieData[i].name === "undefined"
          ? `series${i}`
          : pieData[i].name,
      type: "surface",
      parametric: true,
      wireframe: {
        show: false,
      },
      pieData: pieData[i],
      pieStatus: {
        selected: false,
        hovered: false,
        k: 1 / 10,
      },
    };

    if (typeof pieData[i].itemStyle != "undefined") {
      let itemStyle: any = {};

      typeof pieData[i].itemStyle.color != "undefined"
        ? (itemStyle.color = pieData[i].itemStyle.color)
        : null;
      typeof pieData[i].itemStyle.opacity != "undefined"
        ? (itemStyle.opacity = pieData[i].itemStyle.opacity)
        : null;

      seriesItem.itemStyle = itemStyle;
    }
    series.push(seriesItem);
  }

  // 使用上一次遍历时，计算出的数据和 sumValue，调用 getParametricEquation 函数，
  // 向每个 series-surface 传入不同的参数方程 series-surface.parametricEquation，也就是实现每一个扇形。
  for (let i = 0; i < series.length; i++) {
    endValue = startValue + series[i].pieData.value;

    series[i].pieData.startRatio = startValue / sumValue;
    series[i].pieData.endRatio = endValue / sumValue;
    series[i].parametricEquation = getParametricEquation(
      series[i].pieData.startRatio,
      series[i].pieData.endRatio,
      false,
      false,
      k,
      series[i].pieData.value
    );

    startValue = endValue;

    legendData.push(series[i].name);
  }

  // // 补充一个透明的圆环，用于支撑高亮功能的近似实现。
  series.push({
    name: "mouseoutSeries",
    type: "surface",
    parametric: true,
    wireframe: {
      show: false,
    },
    itemStyle: {
      opacity: 0.1,
      color: "#E1E8EC",
    },
    parametricEquation: {
      u: {
        min: 0,
        max: Math.PI * 2,
        step: Math.PI / 20,
      },
      v: {
        min: 0,
        max: Math.PI,
        step: Math.PI / 20,
      },
      x: function (u: any, v: any) {
        return ((Math.sin(v) * Math.sin(u) + Math.sin(u)) / Math.PI) * 2;
      },
      y: function (u: any, v: any) {
        return ((Math.sin(v) * Math.cos(u) + Math.cos(u)) / Math.PI) * 2;
      },
      z: function (u: any, v: any) {
        return Math.cos(v) > 0 ? -0.5 : -5;
      },
    },
  });

  // // 补充一个透明的圆环，用于支撑高亮功能的近似实现。
  series.push({
    name: "mouseoutSeries",
    type: "surface",
    parametric: true,
    wireframe: {
      show: false,
    },
    itemStyle: {
      opacity: 0.1,
      color: "#E1E8EC",
    },
    parametricEquation: {
      u: {
        min: 0,
        max: Math.PI * 2,
        step: Math.PI / 20,
      },
      v: {
        min: 0,
        max: Math.PI,
        step: Math.PI / 20,
      },
      x: function (u: any, v: any) {
        return ((Math.sin(v) * Math.sin(u) + Math.sin(u)) / Math.PI) * 2;
      },
      y: function (u: any, v: any) {
        return ((Math.sin(v) * Math.cos(u) + Math.cos(u)) / Math.PI) * 2;
      },
      z: function (u: any, v: any) {
        return Math.cos(v) > 0 ? -5 : -7;
      },
    },
  });
  series.push({
    name: "mouseoutSeries",
    type: "surface",

    parametric: true,
    wireframe: {
      show: false,
    },
    itemStyle: {
      opacity: 0.1,
      color: "#E1E8EC",
    },
    parametricEquation: {
      u: {
        min: 0,
        max: Math.PI * 2,
        step: Math.PI / 20,
      },
      v: {
        min: 0,
        max: Math.PI,
        step: Math.PI / 20,
      },
      x: function (u: any, v: any) {
        return (
          ((Math.sin(v) * Math.sin(u) + Math.sin(u)) / Math.PI) * 2.2
        );
      },
      y: function (u: any, v: any) {
        return (
          ((Math.sin(v) * Math.cos(u) + Math.cos(u)) / Math.PI) * 2.2
        );
      },
      z: function (u: any, v: any) {
        return Math.cos(v) > 0 ? -7 : -7;
      },
    },
  });

  // 准备待返回的配置项，把准备好的 legendData、series 传入。
  let option = {
    //animation: false,

    legend: {
      //left: '50%',
      //top: 'center',
      textStyle: {
        fontSize: 18,
      },
      // icon:'diamond',
      data: legendData,
      formatter: (params: any) => {
        return params;
      },
    },
    tooltip: {
      formatter: (params: any) => {
        if (params.seriesName !== "mouseoutSeries") {
          return `${params.seriesName
            }<br/><span style="display:inline-block;margin-right:5px;border-radius:10px;width:10px;height:10px;background-color:${params.color
            };"></span>${option.series[params.seriesIndex].pieData.value}`;
        }
        return "";
      },
    },
    xAxis3D: {},
    yAxis3D: {},
    zAxis3D: {},
    grid3D: {
      viewControl: {
        autoRotate: true,
      },
      left: "center",
      width: "50%",
      height: "auto",
      show: false,
      boxHeight: 60,
    },

    series: series,
  };
  return option;
}

watch(() => toRaw(props.option), (newValue, _oldValue) => {
  console.log(111);
  let options: any = getPie3D(
    [
      {
        name: "性能测试",
        value: 134,
      },
      {
        name: "安全",
        value: 56,
      },
      {
        name: "功能",
        value: 57,
      },
      {
        name: "易用性",
        value: 51,
      },
      {
        name: "代码",
        value: 11,
      },
    ],
    0.7
  );
  console.log('options', options);
  // options = newValue
  newOptions.value = options;
}, { immediate: true })

onMounted(async () => {
  // const data2 = await fetch();
  // newOptions.value = data2 as any[];
});


// 通过给 ECharts title 注册鼠标 over 与 out 事件，来控制 yAxis.name 的 color，从而达到 “口径” 的显示与隐藏功能
const onMouseover = (_: object, chart: any, option: any) => {
  // (option.yAxis).nameTextStyle.color = '#000';
  chart.setOption(option);
};

const onMouseout = (_: object, chart: any, option: any) => {
  // (option.yAxis).nameTextStyle.color = 'transparent';
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
