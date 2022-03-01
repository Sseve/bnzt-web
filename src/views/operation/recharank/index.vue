<template>
  <div class="parent-box">
    <el-date-picker
      v-model="value"
      type="daterange"
      size="small"
      align="right"
      unlink-panels
      range-separator="至"
      start-placeholder="开始日期"
      end-placeholder="结束日期"
      :picker-options="pickerOptions">
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
    pickerOptions: {
          shortcuts: [{
            text: '最近一周',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近一个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近三个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit('pick', [start, end]);
            }
          }]
        },
      value: '',
      tableData: [],
      zone: '',
      page: {
        size: 10,
        num: 1
      },
      total: 0,
      listLoading: false
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