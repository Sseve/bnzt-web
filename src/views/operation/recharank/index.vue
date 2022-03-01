<template>
  <div class="parent-box">
    <el-date-picker
      v-model="value"
      type="daterange"
      size="small"
      align="right"
      unlink-panels
      format="yyyy-MM-dd"
      range-separator="至"
      start-placeholder="开始日期"
      end-placeholder="结束日期"
      value-format="yyyy-MM-dd">>
    </el-date-picker>
    <el-input v-model="zone" size="small" placeholder="区服" style="width:120px;margin-left:10px"></el-input>
    <el-button type="primary" size="small" icon="el-icon-search" style="margin-left:10px" @click="recharank">搜索</el-button>
    <el-table
    :data="tableData"
    style="width: 100%"
    :row-class-name="tableRowClassName">
    <el-table-column
        prop="id"
        label="区服ID"
        width="auto">
    </el-table-column>
    <el-table-column
        prop="uid"
        label="玩家ID"
        width="auto">
    </el-table-column>
    <el-table-column
        prop="chargeNum"
        label="充值金额"
        width="auto">
    </el-table-column>
    <el-table-column
        prop="chargeTime"
        label="最后充值时间"
        width="auto">
    </el-table-column>
    </el-table>
     <!-- 分页 -->
    <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
      :current-page="page.num" :page-sizes="[10, 15, 20, 50, 100]" :page-size="page.size"
      layout="total, sizes, prev, pager, next, jumper" :total="total">
    </el-pagination>
  </div>
</template>

<script>
import { Recharank } from '@/api/operation'
export default {
data() {
  return {
    value: '',
    tableData: [],
    zone: '',
    listLoading: false,
    page: {
      size: 10,
      num: 1
    },
    total: 0
  }
},
methods: {
  handleClose(done) {
    done()
  },
  tableRowClassName({row, rowIndex}) {
    if (rowIndex % 2 === 1) {
      return 'warning-row';
    } else if (rowIndex % 2 === 0) {
      return 'success-row';
    }
    return '';
  },
  handleSizeChange(newSize) {
    this.page.size = newSize
    this.recharank()
  },
  handleCurrentChange(newPage) {
    this.page.num = newPage
    this.recharank()
  },
  recharank() {
    const data = {stime: this.value[0], etime: this.value[1], zone: this.zone, page: this.page}
    this.listLoading = true
    Recharank(data).then(response => {
      console.log(response.data)
      if (response.code === 200 && response.data.length !== 0) {
        this.tableData = []
        response.data.hits.hits.forEach(v => {
          this.tableData.push({
            id: v._source.properties.server_id,
            uid: v._source.properties.uid,
            chargeNum: v._source.properties.pay_amount,
            chargeTime: v._source['#time']
          })
        })
        this.total = response.data.hits.total.value
        this.listLoading = false
      }
    })
  }
}
}
</script>

<style scoped>
.parent-box {
    margin: 40px;
}
.el-table .warning-row {
  background: oldlace;
}

.el-table .success-row {
  background: #f0f9eb;
}
</style>