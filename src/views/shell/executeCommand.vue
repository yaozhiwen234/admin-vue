<template>
  <div class="app-container">
    <date-range-picker
      v-model="date"
      size="small"
      style="width: 250px;"
      format="yyyy-MM-dd"
      value-format="yyyy-MM-dd"
      start-placeholder="开始日期"
      end-placeholder="结束日期"
      class="date-item"
    />
    <el-input v-model="listQuery.ip" size="small" style="width: 150px;" prefix-icon="el-icon-search" class="filter-item" placeholder="请输入IP" clearable />
    <el-input v-model="listQuery.shellDescribe" size="small" style="width: 150px;" prefix-icon="el-icon-search" class="filter-item" placeholder="请输入分组名称" clearable />

    <el-button size="small" class="filter-item" style="width: 120px;" type="primary" @click="getExecuteCommand()">查询</el-button>
    <el-button size="small" class="filter-item" style="width: 120px;" type="success" @click="dialogVisible = true">执行命令</el-button>
    <el-table ref="table" :data="list" style="width: 100%;">
      <el-table-column type="selection" min-width="1%" />
      <el-table-column
        header-align="center"
        align="center"
        label="序号"
        width="50"
      >
        <template slot-scope="scope">
          <!--                          type="index"-->
          {{ (listQuery.pageNumber - 1) * listQuery.pageSize + scope.$index + 1 }}
        </template>
      </el-table-column>
      <el-table-column :show-overflow-tooltip="true" prop="shellDescribe" label="分组" min-width="30%" />
      <el-table-column :show-overflow-tooltip="true" prop="shellHost" label="host" min-width="20%" />
      <el-table-column :show-overflow-tooltip="true" prop="shellUserName" label="登录用户名" min-width="15%" />
      <el-table-column :show-overflow-tooltip="true" prop="shellUserPassword" label="登陆密码" min-width="25%" />
      <el-table-column :show-overflow-tooltip="true" prop="shellCommand" label="执行命令" min-width="30%" />
      <el-table-column :show-overflow-tooltip="true" prop="statusVal" label="执行状态" min-width="15%" />
      <el-table-column :show-overflow-tooltip="true" prop="createTime" label="创建日期" min-width="25%" />
      <el-table-column min-width="25%" prop="updateTime" label="编辑时间" />

      <el-table-column label="操作" min-width="25%" fixed="right">
        <template slot-scope="scope">
          <el-button type="success" class="filter-item" style="width: 80px;" size="small" @click="excuteCommand(scope.row)">重新执行</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog
      title="执行命令"
      :visible.sync="dialogVisible"
      width="30%"
    >
      <el-form ref="shellDescribe" :model="execute" label-width="100px" class="demo-ruleForm">
        <el-form-item label="执行分组">
          <el-select v-model="execute.shellBaseId" size="small" style="width: 150px;" clearable filterable placeholder="请选择执行分组">
            <el-option
              v-for="item in shellBaseList"
              :key="item.id"
              :label="item.shellDescribe"
              :value="item.id"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="ip">
          <el-input v-model="execute.ips" type="textarea" />
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="excuteCommand()">确 定</el-button>
      </span>
    </el-dialog>

    <!--分页组件-->
    <div class="grid-content bg-purple" style="margin-top: -30px;">
      <pagination
        v-show="total > 0"
        :total="total"
        :page.sync="listQuery.pageNumber"
        :limit.sync="listQuery.pageSize"
        @pagination="getExecuteCommand"
      />
    </div>

  </div>
</template>

<script>
import DateRangePicker from '@/components/DateRangePicker'
import Pagination from '@/components/Pagination' // secondary package based on el-pagination
import request from '@/utils/request'
import qs from 'qs'
export default {
  name: 'Shell',
  components: { DateRangePicker, Pagination },
  data() {
    return {
      dialogVisible: false,
      execute: {
        shellBaseId: null,
        ips: null
      },
      date: [],
      list: null,
      shellBaseList: null,
      total: 0,
      listQuery: {
        pageNumber: 1,
        pageSize: 20,
        shellDescribe: null,
        ip: null,
        startTime: null,
        endTime: null
      }
    }
  },
  created() {
    this.getList()
    this.getExecuteCommand()
  },
  methods: {
    getList() {
      request({
        url:
            '/api/shellBase/getShellBase?' +
            qs.stringify(this.listQuery, { indices: false }),
        method: 'get'
      })
        .then(response => {
          this.shellBaseList = response.data
        })
        .catch(error => {
          console.log(error)
        })
    },
    excuteCommand(row) {
      if (row && row.shellHost) {
        this.execute.shellBaseId = row.shellBaseId
        this.execute.ips = row.shellHost
      }
      request({
        url:
            '/api/shellBase/executeShell?' +
            qs.stringify(this.execute, { indices: false }),
        method: 'get'
      })
        .then(response => {
          this.execute.shellBaseId = null
          this.execute.ips = null
          this.dialogVisible = false
          this.$notify.success({
            title: response.timestamp,
            message: response.msg
          })
        })
        .catch(error => {
          console.log(error)
        })
    },
    getExecuteCommand() {
      if (this.date && this.date.length === 2) {
        this.listQuery.startTime = this.date[0]
        this.listQuery.endTime = this.date[1]
      } else {
        this.listQuery.startTime = ''
        this.listQuery.endTime = ''
        this.date = []
      }
      request({
        url:
            '/api/shellBase/getExecuteCommand?' +
            qs.stringify(this.listQuery, { indices: false }),
        method: 'get'
      })
        .then(response => {
          this.total = response.total
          this.list = response.record
        })
        .catch(error => {
          console.log(error)
        })
    },
    handleFilter() {
      this.listQuery.pageNumber = 1
      this.getList()
    }

  }

}
</script>

