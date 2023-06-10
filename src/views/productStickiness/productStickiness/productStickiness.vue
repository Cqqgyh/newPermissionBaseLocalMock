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
    <!--    直访率-->
    <DirectVisitRate ref="directVisitRateRef"></DirectVisitRate>
    <!--    跳出率-->
    <BounceRate ref="bounceRateRef"></BounceRate>
    <!--    人均PV-->
    <PagesPerVisit ref="pagesPerVisitRef"></PagesPerVisit>
    <!--    平均访问时长-->
    <AverageVisitTime ref="averageVisitTimeRef"></AverageVisitTime>
    <!--    用户生命周期-->
    <UserLifeTime ref="userLifeTimeRef"></UserLifeTime>
    <!--    最高同时在线人数-->
    <MaxConcurrentUsers ref="maxConcurrentUsersRef"></MaxConcurrentUsers>
  </el-card>
</template>
<script setup lang="ts">
import { ref, watch } from 'vue'
import { dayjs } from 'element-plus'
import { ShowType, ShowTypeMap } from '@/enums/constEnums'
import 'echarts'
import DirectVisitRate from '@/views/productStickiness/productStickiness/components/directVisitRate.vue'
import BounceRate from '@/views/productStickiness/productStickiness/components/bounceRate.vue'
import PagesPerVisit from '@/views/productStickiness/productStickiness/components/pagesPerVisit.vue'
import AverageVisitTime from '@/views/productStickiness/productStickiness/components/averageVisitTime.vue'
import UserLifeTime from '@/views/productStickiness/productStickiness/components/userLifeTime.vue'
import MaxConcurrentUsers from '@/views/productStickiness/productStickiness/components/maxConcurrentUsers.vue'
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
// 直访率
const directVisitRateRef = ref<InstanceType<typeof DirectVisitRate>>()
// 跳出率
const bounceRateRef = ref<InstanceType<typeof BounceRate>>()
// 人均PV
const pagesPerVisitRef = ref<InstanceType<typeof PagesPerVisit>>()
// 平均访问时长
const averageVisitTimeRef = ref<InstanceType<typeof AverageVisitTime>>()
// 用户生命周期
const userLifeTimeRef = ref<InstanceType<typeof UserLifeTime>>()
// 最高同时在线人数
const maxConcurrentUsersRef = ref<InstanceType<typeof MaxConcurrentUsers>>()
//#endregion
// 监视searchParams的变化 修改子组件查询参数
watch(
  searchParams,
  () => {
    // 修改子组件差选参数
    directVisitRateRef.value?.setSearchParams(searchParams.value)
    bounceRateRef.value?.setSearchParams(searchParams.value)
    pagesPerVisitRef.value?.setSearchParams(searchParams.value)
    averageVisitTimeRef.value?.setSearchParams(searchParams.value)
    userLifeTimeRef.value?.setSearchParams(searchParams.value)
    maxConcurrentUsersRef.value?.setSearchParams(searchParams.value)
  },
  { deep: true },
)
</script>

<style lang="scss" scoped></style>
