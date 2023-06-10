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
import { ShowType } from '@/enums/constEnums'
import 'echarts'
import * as echarts from 'echarts'
import VChart from 'vue-echarts'
import { RegionalDistributionInterfaceRes } from '@/api/userAnalysis/types'
import { getRegionalDistribution } from '@/api/userAnalysis'
// 引入中国地图
import chinaMap from './chinaMap.json'
echarts.registerMap('china', chinaMap as any)
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
const pageData = ref({} as RegionalDistributionInterfaceRes)
// 分析pageData数据
const pageDataAnalyse = computed(() => {
  let data = pageData.value.data || []
  // 计算value总量
  const totalValue = data.reduce((total, item) => {
    return total + item.value
  }, 0)
  // 计算每个value所占的百分比，并添加进data
  data = data.map((item) => {
    return {
      ...item,
      percent: parseFloat(((item.value / totalValue) * 100).toFixed(2)),
    }
  })
  // 取出value最大的前9个数据
  const sortPageData = data
    .sort((a, b) => {
      return b.value - a.value
    })
    .slice(0, 10)
  return {
    sortPageData,
    data,
  }
})
// 计算Y轴数据
const yData = computed(() => {
  return (
    pageDataAnalyse.value.sortPageData?.map(
      (item, index) => index + item.name,
    ) || []
  )
})
// 计算柱状图数据
const barData = computed(() => {
  return pageDataAnalyse.value.sortPageData?.map((item) => item) || []
})
// 展示数据图表配置
const option = ref({
  title: [
    {
      show: true,
      text: '排名情况',
      textStyle: {
        color: '#2D3E53',
        fontSize: 18,
      },
      right: 180,
      top: 100,
    },
  ],
  tooltip: {
    show: true,
    formatter: function (params: any) {
      return `${params.name}：${params.data?.value}（${params.data?.percent}%）`
    },
  },
  visualMap: {
    type: 'continuous',
    orient: 'horizontal',
    itemWidth: 10,
    itemHeight: 80,
    text: ['高', '低'],
    showLabel: true,
    seriesIndex: [0],
    min: 0,
    max: 40,
    inRange: {
      color: ['#6FCF6A', '#FFFD64', '#FF5000'],
    },
    textStyle: {
      color: '#7B93A7',
    },
    bottom: 30,
    left: 'left',
  },
  grid: {
    right: 100,
    top: 135,
    bottom: 100,
    width: '20%',
  },
  xAxis: {
    show: false,
  },
  yAxis: {
    type: 'category',
    inverse: true,
    nameGap: 16,
    axisLine: {
      show: false,
      lineStyle: {
        color: '#ddd',
      },
    },
    axisTick: {
      show: false,
      lineStyle: {
        color: '#ddd',
      },
    },
    axisLabel: {
      interval: 0,
      margin: 85,
      color: '#455A74',
      align: 'left',
      fontSize: 14,
      rich: {
        a: {
          color: '#fff',
          backgroundColor: '#FAAA39',
          width: 20,
          height: 20,
          align: 'center',
          borderRadius: 2,
        },
        b: {
          color: '#fff',
          backgroundColor: '#4197FD',
          width: 20,
          height: 20,
          align: 'center',
          borderRadius: 2,
        },
      },
      formatter: function (params: any) {
        if (parseInt(params.slice(0, 1)) < 3) {
          return [
            '{a|' +
              (parseInt(params.slice(0, 1)) + 1) +
              '}' +
              '  ' +
              params.slice(1),
          ].join('\n')
        } else {
          return [
            '{b|' +
              (parseInt(params.slice(0, 1)) + 1) +
              '}' +
              '  ' +
              params.slice(1),
          ].join('\n')
        }
      },
    },
    data: yData,
  },
  geo: {
    // roam: true,
    map: 'china',
    left: '100',
    // right: '400',
    // layoutSize: '80%',
    zoom: 1.2, // 将地图缩小到原来的80%
    emphasis: {
      itemStyle: {
        areaColor: '#fff464',
      },
      label: {
        show: false,
      },
    },
  },
  series: [
    {
      name: 'mapSer',
      type: 'map',
      roam: false,
      geoIndex: 0,
      label: {
        show: false,
      },
      data: computed(() => {
        return pageDataAnalyse.value.data || []
      }),
    },
    {
      name: 'barSer',
      type: 'bar',
      roam: false,
      visualMap: false,
      zlevel: 2,
      barMaxWidth: 8,
      barGap: 0,
      itemStyle: {
        color: function (params: any) {
          // build a color map as your need.
          let colorList = [
            {
              colorStops: [
                {
                  offset: 0,
                  color: '#FFD119', // 0% 处的颜色
                },
                {
                  offset: 1,
                  color: '#FFAC4C', // 100% 处的颜色
                },
              ],
            },
            {
              colorStops: [
                {
                  offset: 0,
                  color: '#00C0FA', // 0% 处的颜色
                },
                {
                  offset: 1,
                  color: '#2F95FA', // 100% 处的颜色
                },
              ],
            },
          ]
          if (params.dataIndex < 3) {
            return colorList[0]
          } else {
            return colorList[1]
          }
        },
        borderRadius: 15,
      },
      data: barData,
      label: {
        show: true,
        position: 'right',
        formatter: function (params: any) {
          return `${params.data?.value}（${params.data?.percent}%）`
        },
        color: '#455A74',
        fontSize: 14,
      },
    },
  ],
})
//#endregion
// 监视searchParams的变化 重新请求数据
watch(
  searchParams,
  () => {
    getRegionalDistributionHandle()
  },
  { deep: true },
)

async function getRegionalDistributionHandle() {
  try {
    const res = await getRegionalDistribution(searchParams.value)
    pageData.value = res.data
  } catch (error) {
    console.log(error)
    ElMessage.error((error as any)?.message || 'Has Error')
  }
}
onMounted(() => {
  getRegionalDistributionHandle()
})
// 对付组件暴露setSearchParams方法
defineExpose({
  setSearchParams,
})
</script>

<style lang="scss" scoped></style>
