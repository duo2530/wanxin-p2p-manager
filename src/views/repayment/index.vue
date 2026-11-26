<template>
  <div class="app-container">
    <div class="filter-row">
      <el-input v-model="projectNo" placeholder="标的编号(精确匹配)" clearable style="width: 260px"
        @keyup.enter.native="search" />
      <el-button type="primary" @click="search">查询</el-button>
      <span v-if="mode === 'today'" class="tip">应还日期:{{ today }}</span>
    </div>
    <el-table v-loading="listLoading" :data="list" element-loading-text="Loading" border fit highlight-current-row>
      <el-table-column align="center" label="标的编号" prop="projectNo" min-width="200" />
      <el-table-column align="center" label="借款人编号" prop="userNo" min-width="200" />
      <el-table-column align="center" label="期数" prop="numberOfPeriods" width="70" />
      <el-table-column align="center" label="应还日期" prop="shouldRepaymentDate" width="110" />
      <el-table-column align="center" label="应还本金(元)" prop="principal" width="110" />
      <el-table-column align="center" label="应还利息(元)" prop="interest" width="110" />
      <el-table-column align="center" label="应还本息(元)" prop="amount" width="110" />
      <el-table-column align="center" label="状态" prop="repaymentStatusName" width="90" />
      <el-table-column align="center" label="实际还款时间" width="160">
        <template slot-scope="scope">
          <span>{{ scope.row.repaymentDate || '-' }}</span>
        </template>
      </el-table-column>
    </el-table>
    <div class="block pageList">
      <el-pagination background layout="total, prev, pager, next" :current-page="pageNo" :page-size="pageSize"
        :total="total" @current-change="handleCurrentChange" />
    </div>
  </div>
</template>

<script>
  import { getRepaymentPlans } from '@/api/api'

  function localToday() {
    const d = new Date()
    const m = String(d.getMonth() + 1).padStart(2, '0')
    const day = String(d.getDate()).padStart(2, '0')
    return `${d.getFullYear()}-${m}-${day}`
  }

  export default {
    name: 'RepaymentPlanList',
    props: {
      // today 当日待还款 / pending 待还款借款 / inprogress 还款中借款 / done 已还款借款
      mode: { type: String, default: 'pending' }
    },
    data() {
      return {
        list: [],
        listLoading: false,
        pageNo: 1,
        pageSize: 10,
        total: 0,
        projectNo: '',
        today: localToday()
      }
    },
    watch: {
      // 同一组件在不同菜单间切换时复用实例,需要重新加载
      mode() {
        this.projectNo = ''
        this.pageNo = 1
        this.today = localToday()
        this.fetchData()
      }
    },
    created() {
      this.fetchData()
    },
    methods: {
      buildQuery() {
        const q = {}
        if (this.mode === 'today') {
          q.repaymentStatus = '0'
          q.shouldRepaymentDate = this.today
        } else if (this.mode === 'pending') {
          q.repaymentStatus = '0'
        } else if (this.mode === 'inprogress') {
          q.inProgress = true
        } else if (this.mode === 'done') {
          q.repaymentStatus = '1'
        }
        const no = this.projectNo.trim()
        if (no) q.projectNo = no
        return q
      },
      fetchData() {
        this.listLoading = true
        getRepaymentPlans(this.pageNo, this.pageSize, this.buildQuery()).then(response => {
          if (response.code === 0) {
            this.list = response.result.content
            this.total = Number(response.result.total)
          } else {
            this.list = []
            this.total = 0
            this.$message({ type: 'info', message: response.msg })
          }
          this.listLoading = false
        }).catch(() => {
          this.list = []
          this.total = 0
          this.listLoading = false
        })
      },
      search() {
        this.pageNo = 1
        this.fetchData()
      },
      handleCurrentChange(val) {
        this.pageNo = val
        this.fetchData()
      }
    }
  }
</script>
<style>
  .filter-row {
    margin-bottom: 12px;
  }
  .filter-row .tip {
    margin-left: 16px;
    color: #909399;
  }
  .pageList {
    text-align: right;
    margin-top: 10px;
  }
</style>
