<template>
  <el-card class="m-t-20">
    <template #header v-if="props.pageConfig?.pageTitle">
      <div class="font-20-bold">{{ props.pageConfig.pageTitle }}</div>
    </template>
    <!--      时间选择-->
    <div v-if="false">
      <span>时间：</span>
      <el-date-picker
        v-model="searchParams.dateRange"
        type="datetimerange"
        start-placeholder="开始时间"
        end-placeholder="结束时间"
        :disabled-date="disabledDate"
        format="YYYY-MM-DD"
        value-format="YYYY-MM-DD"
        :shortcuts="shortcuts"
        :clearable="false"
      />
    </div>
    <!--      图表-->
    <v-chart style="height: 50vh" :option="option" autoresize />
  </el-card>
</template>
<script setup lang="ts">
import { computed, onMounted, PropType, ref, watch } from 'vue'
import { dayjs, ElMessage } from 'element-plus'
import { ShowType } from '@/enums/constEnums'
import 'echarts'
import VChart from 'vue-echarts'
import { BrowserInterfaceRes } from '@/api/userAnalysis/types'
import { getBrowser } from '@/api/userAnalysis'
interface PageConfig {
  // 页面标题
  pageTitle?: string
}
const props = defineProps({
  pageConfig: {
    type: Object as PropType<PageConfig>,
  },
})

//#region <表单相关>
// 搜索参数
const searchParams = ref({
  // 时间范围，默认值 最近7天
  dateRange: [
    dayjs().subtract(7, 'day').format('YYYY-MM-DD'),
    dayjs().format('YYYY-MM-DD'),
  ],
  // 汇总 PC WAP绑定的值  默认值为汇总
  type: ShowType.All,
})
// 修改搜索参数的方法
const setSearchParams = (value: typeof searchParams.value) => {
  searchParams.value = JSON.parse(JSON.stringify(value))
}
// 日期选择快捷选项
const shortcuts = [
  {
    text: '今天',
    value: [dayjs().format('YYYY-MM-DD'), dayjs().format('YYYY-MM-DD')],
  },
  {
    text: '昨天',
    value: [
      dayjs().subtract(1, 'day').format('YYYY-MM-DD'),
      dayjs().subtract(1, 'day').format('YYYY-MM-DD'),
    ],
  },
  {
    text: '本周',
    value: [
      // 本周周一
      dayjs()
        .subtract(dayjs().day() ? dayjs().day() - 1 : 6, 'day')
        .format('YYYY-MM-DD'),
      // 今天
      dayjs().format('YYYY-MM-DD'),
    ],
  },
  {
    text: '上周',
    value: [
      // 上周周一
      dayjs()
        .subtract(dayjs().day() ? dayjs().day() - 1 : 6, 'day')
        .subtract(7, 'day')
        .format('YYYY-MM-DD'),
      // 上周周日
      dayjs()
        .subtract(dayjs().day() ? dayjs().day() - 1 : 6, 'day')
        .subtract(1, 'day')
        .format('YYYY-MM-DD'),
    ],
  },
  {
    text: '最近7天',
    value: [
      dayjs().subtract(7, 'day').format('YYYY-MM-DD'),
      dayjs().format('YYYY-MM-DD'),
    ],
  },
  {
    text: '本月',
    value: [
      // 本月月初
      dayjs().startOf('month').format('YYYY-MM-DD'),
      // 今天
      dayjs().format('YYYY-MM-DD'),
    ],
  },
  {
    text: '上月',
    value: [
      // 上月月初
      dayjs().subtract(1, 'month').startOf('month').format('YYYY-MM-DD'),
      // 上月月末
      dayjs().subtract(1, 'month').endOf('month').format('YYYY-MM-DD'),
    ],
  },
  {
    text: '最近一个月',
    value: [
      dayjs().subtract(1, 'month').format('YYYY-MM-DD'),
      dayjs().format('YYYY-MM-DD'),
    ],
  },
]
// 禁用时间
const disabledDate = (time: Date) => {
  return time.getTime() > Date.now()
}
//#endregion
//#region <echarts中options数据相关>
// 计算X轴数据
// 数据总览数据
const pageData = ref({} as BrowserInterfaceRes)
// 分析pageData数据
const pageDataAnalyse = computed(() => {
  // 计算数据总和
  let total = pageData.value.data?.reduce((total, item) => {
    return total + item.value
  }, 0)
  let data =
    pageData.value.data
      ?.map((item) => {
        return {
          name: item.name,
          value: item.value,
          percent: parseFloat(((item.value / total) * 100).toFixed(2)),
        }
      })
      .sort((a, b) => b.value - a.value) || []
  return {
    data,
  }
})
// 计算Y轴数据
const yAxisData = computed(() => {
  return pageDataAnalyse.value.data?.map((item) => item.name)
})
// 计算series数据
const seriesData = computed(() => {
  return pageDataAnalyse.value.data?.map((item) => item.percent)
})
// 展示数据图表配置
const option = ref({
  title: {
    text: '',
  },
  tooltip: {
    // 			formatter: '{c} 次',
  },
  grid: {
    left: '3%',
    right: '4%',
    bottom: '3%',
    containLabel: true,
  },
  xAxis: {
    type: 'value',
    show: false,
    splitLine: {
      show: false,
    },
    axisTick: {
      show: false,
    },
    axisLine: {
      lineStyle: {
        color: ['#bababa'],
      },
    },
  },
  yAxis: [
    {
      data: yAxisData,
      inverse: true, //反向
      axisLine: {
        show: false,
      },
      axisTick: {
        show: false,
      },
    },
  ],
  series: [
    {
      name: '数据量',
      type: 'bar',
      barWidth: 20,
      data: seriesData,
      color: {
        type: 'linear',
        x: 0,
        y: 0,
        x2: 1,
        y2: 0,
        colorStops: [
          {
            offset: 1,
            color: '#0fe5e3', // 0% 处的颜色
          },
          {
            offset: 0,
            color: '#2ca1d6', // 100% 处的颜色
          },
        ],
        // global: false, // 缺省为 false
      },
      itemStyle: {
        borderRadius: [30, 30, 30, 30],
      },
      label: {
        show: true,
        formatter: '{c}%', // 展示数据值
        // color: '#fff',
        fontSize: 14,
        position: 'right', // 将标签放置在柱子的右侧
      },
    },
  ],
})
//#endregion
// 监视searchParams的变化 重新请求数据
watch(
  searchParams,
  () => {
    getBrowserHandle()
  },
  { deep: true },
)

async function getBrowserHandle() {
  try {
    const res = await getBrowser(searchParams.value)
    pageData.value = res.data
  } catch (error) {
    console.log(error)
    ElMessage.error((error as any)?.message || 'Has Error')
  }
}
onMounted(() => {
  getBrowserHandle()
})
// 对付组件暴露setSearchParams方法
defineExpose({
  setSearchParams,
})
</script>

<style lang="scss" scoped></style>
