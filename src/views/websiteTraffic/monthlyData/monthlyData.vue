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
    <!--    今日数据总览-->
    <el-card>
      <template #header>
        <div class="font-20-bold">今日数据总览</div>
      </template>
      <el-table
        :data="dataOverview.overviewDataList"
        style="width: 100%"
        border
      >
        <el-table-column prop="title" label=""></el-table-column>
        <el-table-column prop="uv" label="访客数(UV)"></el-table-column>
        <el-table-column prop="pv" label="浏览量(PV)"></el-table-column>
        <el-table-column prop="uid" label="新增登录用户(UID)"></el-table-column>
        <el-table-column
          prop="avgVisitTime"
          label="平均访问时长"
        ></el-table-column>
        <el-table-column
          prop="maxOnline"
          label="最高同时在线人数"
        ></el-table-column>
      </el-table>
    </el-card>
    <!--    流量数据-->
    <el-card class="m-t-20">
      <template #header>
        <div class="font-20-bold">流量数据</div>
      </template>
      <!--      时间选择-->
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
      <!--      图表-->
      <v-chart
        style="height: 50vh"
        class="m-t-40"
        :option="option"
        autoresize
      />
    </el-card>
    <!--    表格-->
    <el-card class="m-t-20">
      <elTableForAllData
        :data="dataOverview.flowDataList"
        style="width: 100%"
        border
      >
        <el-table-column prop="date" label="日期"></el-table-column>
        <el-table-column prop="uv" label="访客数(UV)"></el-table-column>
        <el-table-column prop="pv" label="浏览量(PV)"></el-table-column>
        <el-table-column prop="uid" label="新增登录用户(UID)"></el-table-column>
        <el-table-column
          prop="avgVisitTime"
          label="平均访问时长"
        ></el-table-column>
        <el-table-column
          prop="maxOnline"
          label="最高同时在线人数"
        ></el-table-column>
      </elTableForAllData>
    </el-card>
  </el-card>
</template>
<script setup lang="ts">
import { computed, onMounted, reactive, ref, watch } from 'vue'
import { dayjs, ElMessage } from 'element-plus'
import { DateType, ShowType, ShowTypeMap } from '@/enums/constEnums'
import 'echarts'
import VChart from 'vue-echarts'
import { getWebsiteOverview } from '@/api/websiteOverview'
import { FlowDataList, OverviewDataList } from '@/api/websiteOverview/types'
import ElTableForAllData from '@/components/elTableForAllData/elTableForAllData.vue'
//#region <表单相关>
// 搜索参数
const searchParams = ref({
  // 时间范围，默认值是本月月初第一台你
  dateRange: [
    dayjs().startOf('month').format('YYYY-MM-DD'),
    dayjs().format('YYYY-MM-DD'),
  ],
  // 汇总 PC WAP绑定的值  默认值为汇总
  type: ShowType.All,
  dateType: DateType.Month,
})
// 单选列表
const typeList = ShowTypeMap
// 日期选择快捷选项
const shortcuts = [
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
  {
    text: '最近一年',
    value: [
      dayjs().subtract(1, 'year').format('YYYY-MM-DD'),
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
const seriesUvList = computed(() => {
  return dataOverview.flowDataList.map((item) => item.uv)
})
const seriesPvList = computed(() => {
  return dataOverview.flowDataList.map((item) => item.pv)
})
const seriesUidList = computed(() => {
  return dataOverview.flowDataList.map((item) => item.uid)
})
// 数据总览数据
const dataOverview = reactive({
  // 数据总览
  overviewDataList: [] as OverviewDataList[],
  //   流量数据
  flowDataList: [] as FlowDataList[],
})
// 展示数据图表配置
const option = ref({
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
    data: ['访客数（UV）', '浏览数（PV）', '新增登录用户（UID）'],
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
        return dataOverview.flowDataList.map((item) => item.date)
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
    if (dataOverview.flowDataList.length <= 1) {
      return [
        {
          name: '访客数（UV）',
          type: 'bar',
          barWidth: '25%', // 设置柱子宽度
          data: seriesUvList.value,
        },
        {
          name: '浏览数（PV）',
          type: 'bar',
          barWidth: '25%', // 设置柱子宽度
          data: seriesUvList.value,
        },
        {
          name: '新增登录用户（UID）',
          type: 'bar',
          barWidth: '25%', // 设置柱子宽度
          data: seriesUidList.value,
        },
      ]
    } else {
      return [
        {
          name: '访客数（UV）',
          type: 'line',
          stack: 'Total',
          areaStyle: {},
          emphasis: {
            focus: 'series',
          },
          data: seriesUvList.value,
        },
        {
          name: '浏览数（PV）',
          type: 'line',
          stack: 'Total',
          areaStyle: {},
          emphasis: {
            focus: 'series',
          },
          data: seriesPvList.value,
        },
        {
          name: '新增登录用户（UID）',
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
          data: seriesUidList.value,
        },
      ]
    }
  }),
})
//#endregion
// 监视searchParams的变化 重新请求数据
watch(
  searchParams,
  () => {
    getWebsiteOverviewHandle()
  },
  { deep: true },
)
async function getWebsiteOverviewHandle() {
  try {
    const res = await getWebsiteOverview(searchParams.value)
    dataOverview.overviewDataList = res.data.overviewDataList
    dataOverview.flowDataList = res.data.flowDataList
  } catch (error) {
    console.log(error)
    ElMessage.error((error as any)?.message || 'Has Error')
  }
}
onMounted(() => {
  getWebsiteOverviewHandle()
})
</script>

<style lang="scss" scoped></style>
