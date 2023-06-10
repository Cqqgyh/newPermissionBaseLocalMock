<template>
  <el-card>
    <template #header>
      <div class="card-header">
        <el-radio-group v-model="searchParams.type">
          <el-radio-button
            v-for="item in typeList"
            :key="item.label"
            :label="item.value"
          >
            {{ item.label }}
          </el-radio-button>
        </el-radio-group>
      </div>
    </template>
    <!--      时间选择-->
    <el-card>
      <div>
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
    </el-card>
    <!--    留存数-->
    <el-card class="m-t-20">
      <template #header>
        <div class="font-20-bold">留存数</div>
      </template>
      <!--      图表-->
      <v-chart
        style="height: 50vh"
        class="m-t-40"
        :option="retainedNumberOption"
        autoresize
      />
    </el-card>
    <!--    留存率-->
    <el-card class="m-t-20">
      <template #header>
        <div class="font-20-bold">留存率</div>
      </template>
      <!--      图表-->
      <v-chart
        style="height: 50vh"
        class="m-t-40"
        :option="retainedRateOption"
        autoresize
      />
    </el-card>
    <!--    表格-->
    <el-card class="m-t-20">
      <elTableForAllData :data="retainedDataList" style="width: 100%" border>
        <el-table-column prop="date" label="日期"></el-table-column>
        <el-table-column label="基础指标" align="center">
          <el-table-column label="留存数" align="center">
            <el-table-column
              prop="day1RetainedUserCount"
              label="次日"
            ></el-table-column>
            <el-table-column
              prop="day3RetainedUserCount"
              label="3日"
            ></el-table-column>
            <el-table-column
              prop="day7RetainedUserCount"
              label="7日"
            ></el-table-column>
            <el-table-column
              prop="day14RetainedUserCount"
              label="14日"
            ></el-table-column>
            <el-table-column
              prop="day30RetainedUserCount"
              label="30日"
            ></el-table-column>
          </el-table-column>
          <el-table-column label="留存率" align="center">
            <el-table-column
              prop="day1RetentionRate"
              label="次日"
            ></el-table-column>
            <el-table-column
              prop="day3RetentionRate"
              label="3日"
            ></el-table-column>
            <el-table-column
              prop="day7RetentionRate"
              label="7日"
            ></el-table-column>
            <el-table-column
              prop="day14RetentionRate"
              label="14日"
            ></el-table-column>
            <el-table-column
              prop="day30RetentionRate"
              label="30日"
            ></el-table-column>
          </el-table-column>
        </el-table-column>
      </elTableForAllData>
    </el-card>
  </el-card>
</template>
<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue'
import { dayjs, ElMessage } from 'element-plus'
import { LivenessType, ShowType, ShowTypeMap } from '@/enums/constEnums'
import 'echarts'
import VChart from 'vue-echarts'
import ElTableForAllData from '@/components/elTableForAllData/elTableForAllData.vue'
import { getLivenessInfo } from '@/api/liveness'
import { LivenessDataList } from '@/api/liveness/types'
//#region <表单相关>
// 搜索参数
const searchParams = ref({
  // 时间范围，默认值是最近7天
  dateRange: [
    dayjs().subtract(7, 'day').format('YYYY-MM-DD'),
    dayjs().format('YYYY-MM-DD'),
  ],
  // 汇总 PC WAP绑定的值  默认值为汇总
  type: ShowType.All,
  dateType: LivenessType.Retain,
})
// 单选列表
const typeList = ShowTypeMap
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
    text: '最近7天',
    value: [
      dayjs().subtract(7, 'day').format('YYYY-MM-DD'),
      dayjs().format('YYYY-MM-DD'),
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
//#region <echarts数据相关>
// 活跃度数据
const retainedDataList = ref([] as LivenessDataList[])
// 计算X轴数据
const xSeriesData = computed(() => {
  let targetObj = {
    day1RetainedUserCount: [] as number[],
    day3RetainedUserCount: [] as number[],
    day7RetainedUserCount: [] as number[],
    day14RetainedUserCount: [] as number[],
    day30RetainedUserCount: [] as number[],
    day1RetentionRate: [] as number[],
    day3RetentionRate: [] as number[],
    day7RetentionRate: [] as number[],
    day14RetentionRate: [] as number[],
    day30RetentionRate: [] as number[],
  }
  retainedDataList.value.forEach((item) => {
    Object.keys(targetObj).forEach((key) =>
      (targetObj as any)[key].push((item as any)[key]),
    )
  })
  return targetObj
})

//#region <留存数options数据相关>
// 展示数据图表配置
const retainedNumberOption = ref({
  title: {
    text: '指标',
  },
  tooltip: {
    trigger: 'axis',
    axisPointer: {
      type: 'cross',
      label: {
        backgroundColor: '#6a7985',
      },
    },
  },
  legend: {
    data: ['次日', '3日', '7日', '14日', '30日'],
  },
  toolbox: {
    feature: {
      saveAsImage: {},
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
      boundaryGap: false,
      data: computed(() => {
        return retainedDataList.value.map((item) => item.date)
      }),
    },
  ],
  yAxis: [
    {
      type: 'value',
    },
  ],
  series: computed(() => {
    // 如果仅展示一天数据的话，改为柱状图,否则是折线图
    if (retainedDataList.value.length <= 1) {
      return [
        {
          name: '次日',
          type: 'bar',
          barWidth: '15%', // 设置柱子宽度
          data: xSeriesData.value.day1RetainedUserCount,
        },
        {
          name: '3日',
          type: 'bar',
          barWidth: '15%', // 设置柱子宽度
          data: xSeriesData.value.day3RetainedUserCount,
        },
        {
          name: '7日',
          type: 'bar',
          barWidth: '15%', // 设置柱子宽度
          data: xSeriesData.value.day7RetainedUserCount,
        },
        {
          name: '14日',
          type: 'bar',
          barWidth: '15%', // 设置柱子宽度
          data: xSeriesData.value.day14RetainedUserCount,
        },
        {
          name: '30日',
          type: 'bar',
          barWidth: '15%', // 设置柱子宽度
          data: xSeriesData.value.day30RetainedUserCount,
        },
      ]
    } else {
      return [
        {
          name: '次日',
          type: 'line',
          stack: 'Total',
          areaStyle: {},
          emphasis: {
            focus: 'series',
          },
          data: xSeriesData.value.day1RetainedUserCount,
        },
        {
          name: '3日',
          type: 'line',
          stack: 'Total',
          areaStyle: {},
          emphasis: {
            focus: 'series',
          },
          data: xSeriesData.value.day3RetainedUserCount,
        },
        {
          name: '7日',
          type: 'line',
          stack: 'Total',
          areaStyle: {},
          emphasis: {
            focus: 'series',
          },
          data: xSeriesData.value.day7RetainedUserCount,
        },
        {
          name: '14日',
          type: 'line',
          stack: 'Total',
          label: {
            show: true,
            position: 'top',
          },
          areaStyle: {},
          emphasis: {
            focus: 'series',
          },
          data: xSeriesData.value.day14RetainedUserCount,
        },
        {
          name: '30日',
          type: 'line',
          stack: 'Total',
          label: {
            show: true,
            position: 'top',
          },
          areaStyle: {},
          emphasis: {
            focus: 'series',
          },
          data: xSeriesData.value.day30RetainedUserCount,
        },
      ]
    }
  }),
})
//#endregion

//#region <留存率options数据相关>
// 展示数据图表配置
const retainedRateOption = ref({
  title: {
    text: '指标',
  },
  tooltip: {
    trigger: 'axis',
    axisPointer: {
      type: 'cross',
      label: {
        backgroundColor: '#6a7985',
      },
    },
    valueFormatter: (value: any) => value + '%',
  },
  legend: {
    data: ['次日', '3日', '7日', '14日', '30日'],
  },
  toolbox: {
    feature: {
      saveAsImage: {},
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
      boundaryGap: false,
      data: computed(() => {
        return retainedDataList.value.map((item) => item.date)
      }),
    },
  ],
  yAxis: [
    {
      type: 'value',
    },
  ],
  series: computed(() => {
    // 如果仅展示一天数据的话，改为柱状图,否则是折线图
    if (retainedDataList.value.length <= 1) {
      return [
        {
          name: '次日',
          type: 'bar',
          barWidth: '15%', // 设置柱子宽度
          data: xSeriesData.value.day1RetentionRate,
        },
        {
          name: '3日',
          type: 'bar',
          barWidth: '15%', // 设置柱子宽度
          data: xSeriesData.value.day3RetentionRate,
        },
        {
          name: '7日',
          type: 'bar',
          barWidth: '15%', // 设置柱子宽度
          data: xSeriesData.value.day7RetentionRate,
        },
        {
          name: '14日',
          type: 'bar',
          barWidth: '15%', // 设置柱子宽度
          data: xSeriesData.value.day14RetentionRate,
        },
        {
          name: '30日',
          type: 'bar',
          barWidth: '15%', // 设置柱子宽度
          data: xSeriesData.value.day30RetentionRate,
        },
      ]
    } else {
      return [
        {
          name: '次日',
          type: 'line',
          stack: 'Total',
          areaStyle: {},
          emphasis: {
            focus: 'series',
          },
          data: xSeriesData.value.day1RetentionRate,
        },
        {
          name: '3日',
          type: 'line',
          stack: 'Total',
          areaStyle: {},
          emphasis: {
            focus: 'series',
          },
          data: xSeriesData.value.day3RetentionRate,
        },
        {
          name: '7日',
          type: 'line',
          stack: 'Total',
          areaStyle: {},
          emphasis: {
            focus: 'series',
          },
          data: xSeriesData.value.day7RetentionRate,
        },
        {
          name: '14日',
          type: 'line',
          stack: 'Total',
          label: {
            show: true,
            position: 'top',
          },
          areaStyle: {},
          emphasis: {
            focus: 'series',
          },
          data: xSeriesData.value.day14RetentionRate,
        },
        {
          name: '30日',
          type: 'line',
          stack: 'Total',
          label: {
            show: true,
            position: 'top',
          },
          areaStyle: {},
          emphasis: {
            focus: 'series',
          },
          data: xSeriesData.value.day30RetentionRate,
        },
      ]
    }
  }),
})
//#endregion

//#endregion
// 监视searchParams的变化 重新请求数据
watch(
  searchParams,
  () => {
    getLivenessInfoHandle()
  },
  { deep: true },
)
async function getLivenessInfoHandle() {
  try {
    const res = await getLivenessInfo(searchParams.value)
    retainedDataList.value = res.data
  } catch (error) {
    console.log(error)
    ElMessage.error((error as any)?.message || 'Has Error')
  }
}
onMounted(() => {
  getLivenessInfoHandle()
})
</script>

<style lang="scss" scoped></style>
