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
    <v-chart style="height: 50vh" class="m-t-40" :option="option" autoresize />
  </el-card>
</template>
<script setup lang="ts">
import { computed, onMounted, PropType, ref, watch } from 'vue'
import { dayjs, ElMessage } from 'element-plus'
import { EducationTypeMap, GenderTypeMap, ShowType } from '@/enums/constEnums'
import 'echarts'
import VChart from 'vue-echarts'
import { GenderAgeEducationInterfaceRes } from '@/api/userAnalysis/types'
import { getGenderAgeEducation } from '@/api/userAnalysis'
interface PageConfig {
  // 页面标题
  pageTitle?: string
}
const props = defineProps({
  pageConfig: {
    type: Object as PropType<PageConfig>,
  },
})

const ageConfig = [
  {
    label: '10-19岁',
    value: [10, 19],
  },
  {
    label: '20-29岁',
    value: [20, 29],
  },
  {
    label: '30-39岁',
    value: [30, 39],
  },
  {
    label: '40-49岁',
    value: [40, 49],
  },
  {
    label: '50-59岁',
    value: [50, 59],
  },
  {
    label: '60-69岁',
    value: [60, 69],
  },
  {
    label: '70岁以上',
    value: [70, 200],
  },
]
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
const pageData = ref([] as GenderAgeEducationInterfaceRes[])
// 分析pageData数据
const pageDataAnalyse = computed(() => {
  return {
    genderDate: calculateGender(pageData.value),
    ageData: calculateAge(pageData.value),
    educationData: calculateEducationType(pageData.value),
  }
})

// 展示数据图表配置
const option = ref({
  backgroundColor: '#ffffff',
  title: [
    {
      id: '0',
      text: '性别比例',
      left: '28%',
      top: '0',
      textAlign: 'center',
    },
    {
      id: '1',
      text: '年龄分布',
      left: '78%',
      top: '0',
      textAlign: 'center',
    },
    {
      id: '2',
      text: '学历分布',
      left: '28%',
      top: '50%',
      textAlign: 'center',
    },
  ],
  tooltip: {
    trigger: 'axis',
    valueFormatter: (value: any) => `${value}%`,
  },
  grid: [
    {
      show: false,
      left: '3%',
      top: '5%',
      containLabel: true,
      width: '45%',
      height: '40%',
    },
    {
      show: false,
      left: '53%',
      top: '5%',
      containLabel: true,
      width: '45%',
      height: '40%',
    },
    {
      show: false,
      left: '3%',
      top: '55%',
      containLabel: true,
      width: '45%',
      height: '40%',
    },
  ],
  xAxis: [
    {
      gridIndex: 0,
      type: 'category',
      axisTick: {
        alignWithLabel: true,
      },
      // 计算生成data
      data: computed(() => {
        return pageDataAnalyse.value.genderDate.map((item) => item.label)
      }),
    },
    {
      gridIndex: 1,
      type: 'category',
      axisTick: {
        alignWithLabel: true,
      },
      // 计算生成data
      data: computed(() => {
        return pageDataAnalyse.value.ageData.map((item) => item.label)
      }),
    },
    {
      gridIndex: 2,
      type: 'category',
      axisTick: {
        alignWithLabel: true,
      },
      // 计算生成data
      data: computed(() => {
        return pageDataAnalyse.value.educationData.map((item) => item.label)
      }),
    },
  ],
  yAxis: [
    {
      gridIndex: 0,
      type: 'value',
      axisLabel: {
        show: true,
        interval: 'auto',
        formatter: '{value}%',
      },
    },
    {
      gridIndex: 1,
      axisLabel: {
        show: true,
        interval: 'auto',
        formatter: '{value}%',
      },
    },
    {
      gridIndex: 2,
      axisLabel: {
        show: true,
        interval: 'auto',
        formatter: '{value}%',
      },
    },
  ],
  series: [
    {
      type: 'bar',
      xAxisIndex: 0,
      yAxisIndex: 0,
      barWidth: '25%',
      // 计算生成data
      data: computed(() => {
        return pageDataAnalyse.value.genderDate.map((item) => item.percentage)
      }),
    },
    {
      type: 'bar',
      xAxisIndex: 1,
      yAxisIndex: 1,
      barWidth: '50%',
      // 计算生成data
      // 计算生成data
      data: computed(() => {
        return pageDataAnalyse.value.ageData.map((item) => item.percentage)
      }),
    },
    {
      type: 'bar',
      xAxisIndex: 2,
      yAxisIndex: 2,
      barWidth: '40%',
      // 计算生成data
      data: computed(() => {
        return pageDataAnalyse.value.educationData.map(
          (item) => item.percentage,
        )
      }),
    },
  ],
})
//#endregion
// 监视searchParams的变化 重新请求数据
watch(
  searchParams,
  () => {
    getGenderAgeEducationHandle()
  },
  { deep: true },
)

/**
 * 计算年龄段的人数
 * @param data
 */
function calculateAge(data: GenderAgeEducationInterfaceRes[]) {
  const ageMap = ageConfig.map((item) => {
    return {
      label: item.label,
      value: item.value,
      // 人数
      count: 0,
      // 百分比
      percentage: 0,
    }
  })
  // 总人数
  let total = data.length
  // 总人数为0时，直接返回
  if (total === 0) {
    return ageMap
  }
  // 遍历数据数组，计算各个年龄段的人数
  data.forEach((item) => {
    ageMap.forEach((mapItem) => {
      if (item.age >= mapItem.value[0] && item.age <= mapItem.value[1]) {
        mapItem.count++
      }
    })
  })
  // 计算每个年龄段的百分比
  ageMap.forEach((item) => {
    item.percentage = parseFloat(((item.count / total) * 100).toFixed(2))
  })
  return ageMap
}
/**
 * 计算教育类型的人数
 * @param data
 */
function calculateEducationType(data: GenderAgeEducationInterfaceRes[]) {
  const EducationMap = EducationTypeMap.map((item) => {
    return {
      label: item.label,
      value: item.value,
      // 人数
      count: 0,
      // 百分比
      percentage: 0,
    }
  })
  // 总人数
  let total = data.length
  // 总人数为0时，直接返回
  if (total === 0) {
    return EducationMap
  }
  // 遍历数据数组，计算各个教育类型的人数
  data.forEach((item) => {
    EducationMap.forEach((mapItem) => {
      if (item.educationType === mapItem.value) {
        mapItem.count++
      }
    })
  })
  // 计算每个教育类型的百分比
  EducationMap.forEach((item) => {
    item.percentage = parseFloat(((item.count / total) * 100).toFixed(2))
  })
  return EducationMap
}
/**
 * 计算性别
 * @param data
 */
function calculateGender(data: GenderAgeEducationInterfaceRes[]) {
  const GenderMap = GenderTypeMap.map((item) => {
    return {
      label: item.label,
      value: item.value,
      // 人数
      count: 0,
      // 百分比
      percentage: 0,
    }
  })
  // 总人数
  let total = data.length
  // 总人数为0时，直接返回
  if (total === 0) {
    return GenderMap
  }
  // 遍历数据数组，计算各个教育类型的人数
  data.forEach((item) => {
    GenderMap.forEach((mapItem) => {
      if (item.gender === mapItem.value) {
        mapItem.count++
      }
    })
  })
  // 计算每个教育类型的百分比
  GenderMap.forEach((item) => {
    item.percentage = parseFloat(((item.count / total) * 100).toFixed(2))
  })
  return GenderMap
}

async function getGenderAgeEducationHandle() {
  try {
    const res = await getGenderAgeEducation(searchParams.value)
    pageData.value = res.data
  } catch (error) {
    console.log(error)
    ElMessage.error((error as any)?.message || 'Has Error')
  }
}
onMounted(() => {
  getGenderAgeEducationHandle()
})
// 对付组件暴露setSearchParams方法
defineExpose({
  setSearchParams,
})
</script>

<style lang="scss" scoped></style>
