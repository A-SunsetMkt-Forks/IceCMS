<script setup lang="ts">
import { useDark, useECharts } from "@pureadmin/utils";
import { type PropType, ref, computed, watch, nextTick } from "vue";

const props = defineProps({
  requireData: {
    type: Array as PropType<Array<number>>,
    default: () => []
  },
  questionData: {
    type: Array as PropType<Array<number>>,
    default: () => []
  },
  chartType: {
    type: Number,
    default: 0 // 0: 资源/订单, 1: 用户/文章
  }
});

// 根据chartType动态设置图例和系列名称
const legendData = computed(() => {
  return props.chartType === 0 ? ["资源", "订单"] : ["用户", "文章"];
});

const seriesNames = computed(() => {
  return props.chartType === 0 ?
    { requireName: "资源", questionName: "订单" } :
    { requireName: "用户", questionName: "文章" };
});

const { isDark } = useDark();

const theme = computed(() => (isDark.value ? "dark" : "light"));

const chartRef = ref();
const { setOptions } = useECharts(chartRef, {
  theme
});

watch(
  () => [props.requireData, props.questionData, props.chartType, legendData.value, seriesNames.value],
  async () => {
    await nextTick(); // 确保DOM更新完成后再执行
    setOptions({
      container: ".bar-card",
      color: ["#41b6ff", "#e85f33"],
      tooltip: {
        trigger: "axis",
        axisPointer: {
          type: "none"
        }
      },
      grid: {
        top: "20px",
        left: "50px",
        right: 0
      },
      legend: {
        data: legendData.value,
        textStyle: {
          color: "#606266",
          fontSize: "0.875rem"
        },
        bottom: 0
      },
      xAxis: [
        {
          type: "category",
          data: ["周一", "周二", "周三", "周四", "周五", "周六", "周日"],
          axisLabel: {
            fontSize: "0.875rem"
          },
          axisPointer: {
            type: "shadow"
          }
        }
      ],
      yAxis: [
        {
          type: "value",
          axisLabel: {
            fontSize: "0.875rem"
          },
          splitLine: {
            show: false // 去网格线
          }
          // name: "单位: 个"
        }
      ],
      series: [
        {
          name: seriesNames.value.requireName,
          type: "bar",
          barWidth: 10,
          itemStyle: {
            color: "#41b6ff",
            borderRadius: [10, 10, 0, 0]
          },
          data: props.requireData
        },
        {
          name: seriesNames.value.questionName,
          type: "bar",
          barWidth: 10,
          itemStyle: {
            color: "#e86033ce",
            borderRadius: [10, 10, 0, 0]
          },
          data: props.questionData
        }
      ]
    });
  },
  {
    deep: true,
    immediate: true
  }
);
</script>

<template>
  <div ref="chartRef" style="width: 100%; height: 365px" />
</template>
