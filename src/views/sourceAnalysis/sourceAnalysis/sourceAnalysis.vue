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
    <el-card class="m-t-20">
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
    </el-card>
    <!--    站外来源细分-->
    <linePieChartOfEcharts
      ref="offsiteSourceRef"
      :pageConfig="offsiteSourceRefConfig"
    />
    <!--    站内来源细分-->
    <linePieChartOfEcharts
      ref="inStationSourceRef"
      :pageConfig="inStationSourceRefConfig"
    />
    <!--    UID来源细分-->
    <linePieChartOfEcharts
      ref="uidSourceRef"
      :pageConfig="uidSourceRefConfig"
    />
  </el-card>
</template>
<script setup lang="ts">
import { ref, watch } from 'vue'
import { dayjs } from 'element-plus'
import { ShowType, ShowTypeMap } from '@/enums/constEnums'
import 'echarts'
import linePieChartOfEcharts from '@/views/sourceAnalysis/sourceAnalysis/components/linePieChartOfEcharts.vue'
import {
  getInStationSource,
  getOffsiteSource,
  getUidSource,
} from '@/api/sourceAnalysis'
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
//#region <子组件>

//#region <站外来源细分>
const offsiteSourceRef = ref<InstanceType<typeof linePieChartOfEcharts>>()
const offsiteSourceRefConfig = ref({
  pageTitle: '站外来源细分',
  apiMethod: getOffsiteSource,
  showFieldConfig: [
    {
      name: '搜索引擎UV',
      resKey: 'searchEngineUV',
      color: '#f6e58d',
    },
    {
      name: '微博UV',
      resKey: 'weiboUV',
      color: '#ffbe76',
    },
    {
      name: '微信UV',
      resKey: 'wechatUV',
      color: '#ff7979',
    },
    {
      name: '直访UV',
      resKey: 'directUV',
      color: '#badc58',
    },
    {
      name: '其他UV',
      resKey: 'otherUV',
      color: '#dff9fb',
    },
  ],
})
//#endregion

//#region <站内来源细分>
const inStationSourceRef = ref<InstanceType<typeof linePieChartOfEcharts>>()
const inStationSourceRefConfig = ref({
  pageTitle: '站内来源细分',
  apiMethod: getInStationSource,
  showFieldConfig: [
    {
      name: 'BannerUV',
      resKey: 'searchEngineUV',
      color: '#cd84f1',
    },
    {
      name: '文字链UV',
      resKey: 'textLinkUV',
      color: '#ffcccc',
    },
    {
      name: '功能插件UV',
      resKey: 'functionPluginUV',
      color: '#ff4d4d',
    },
    {
      name: '产品UV',
      resKey: 'productUV',
      color: '#ffaf40',
    },
    {
      name: '其他UV',
      resKey: 'otherUV',
      color: '#fffa65',
    },
  ],
})
//#endregion

//#region <UID来源细分>
const uidSourceRef = ref<InstanceType<typeof linePieChartOfEcharts>>()
const uidSourceRefConfig = ref({
  pageTitle: 'UID来源细分',
  apiMethod: getUidSource,
  showFieldConfig: [
    {
      name: '关注概念',
      resKey: 'followConceptChannel',
      color: '#f6e58d',
    },
    {
      name: '投资易50',
      resKey: 'investEasy50Channel',
      color: '#ffbe76',
    },
    {
      name: '多空热股',
      resKey: 'longShortHotStockChannel',
      color: '#ff7979',
    },
    {
      name: '数策选股',
      resKey: 'dataStrategyStockChannel',
      color: '#badc58',
    },
    {
      name: '大数据诊股',
      resKey: 'bigDataDiagnosisStockChannel',
      color: '#dff9fb',
    },
    {
      name: '其他',
      resKey: 'otherChannel',
      color: '#18dcff',
    },
  ],
})
//#endregion

//#endregion
// 监视searchParams的变化 修改子组件查询参数
watch(
  searchParams,
  () => {
    // 修改子组件差选参数
    offsiteSourceRef.value?.setSearchParams(searchParams.value)
    inStationSourceRef.value?.setSearchParams(searchParams.value)
    uidSourceRef.value?.setSearchParams(searchParams.value)
  },
  { deep: true },
)
</script>

<style lang="scss" scoped></style>
