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
    <v-chart style="height: 50vh" class="m-t-10" :option="option" autoresize />
  </el-card>
</template>
<script setup lang="ts">
import { computed, onMounted, PropType, ref, watch } from 'vue'
import { dayjs, ElMessage } from 'element-plus'
import { ShowType } from '@/enums/constEnums'
import 'echarts'
import VChart from 'vue-echarts'
import { InterestIndexInterfaceRes } from '@/api/userAnalysis/types'
import { getInterestIndex } from '@/api/userAnalysis'
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
// 数据总览数据
const pageData = ref({} as InterestIndexInterfaceRes)
// 计算X轴数据
const computedXAxis = computed(() => {
  return pageData.value.data?.map((item) => item.label) || []
})
// 计算Y轴数据
const computedYSeries = computed(() => {
  return pageData.value.data?.map((item) => item.value) || []
})
// 展示数据图表配置
const option = ref({
  title: {
    text: '',
  },
  color: ['#4fa8f9'],
  tooltip: {
    trigger: 'axis',
    axisPointer: {
      type: 'cross',
      label: {
        backgroundColor: '#6a7985',
      },
    },
  },
  grid: {
    left: '3%',
    right: '4%',
    bottom: '3%',
    containLabel: true,
  },
  xAxis: [
    {
      type: 'category',
      axisTick: {
        alignWithLabel: true,
      },
      axisLabel: {
        interval: 0, //横轴信息全部显示
        rotate: -30, //-30度角倾斜显示
      },
      data: computedXAxis,
    },
  ],
  yAxis: [
    {
      type: 'value',
    },
  ],
  series: [
    {
      type: 'bar',
      barWidth: '80%', // 设置柱子宽度
      data: computedYSeries,
    },
  ],
})
//#endregion
// 监视searchParams的变化 重新请求数据
watch(
  searchParams,
  () => {
    getInterestIndexHandle()
  },
  { deep: true },
)
async function getInterestIndexHandle() {
  try {
    const res = await getInterestIndex(searchParams.value)
    pageData.value = res.data
  } catch (error) {
    console.log(error)
    ElMessage.error((error as any)?.message || 'Has Error')
  }
}
onMounted(() => {
  getInterestIndexHandle()
})
// 对付组件暴露setSearchParams方法
defineExpose({
  setSearchParams,
})
</script>

<style lang="scss" scoped></style>
