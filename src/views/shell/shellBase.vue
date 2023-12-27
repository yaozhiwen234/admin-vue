<template>
  <div class="app-container">
    <el-input v-model="listQuery.shellDescribe" size="small" style="width: 150px;" prefix-icon="el-icon-search" class="filter-item" placeholder="请输入标题" clearable />
    <el-button size="small" class="filter-item" style="width: 120px;" type="success" @click="handleFilter()">查询</el-button>
    <el-button size="small" class="filter-item" style="width: 120px;" type="primary" @click="saveOrUpdate()">新建执行分组</el-button>
    <el-table ref="table" :data="list" style="width: 100%;">
      <el-table-column type="selection" min-width="1%" />
      <el-table-column :show-overflow-tooltip="true" prop="id" label="id" min-width="15%" />

      <el-table-column :show-overflow-tooltip="true" prop="shellDescribe" label="分组" min-width="15%" />
      <el-table-column :show-overflow-tooltip="true" prop="shellUserName" label="登录用户名" min-width="15%" />
      <el-table-column :show-overflow-tooltip="true" prop="shellUserPassword" label="登陆密码" min-width="15%" />
      <el-table-column :show-overflow-tooltip="true" prop="shellCommand" label="执行命令" min-width="15%" />
      <el-table-column :show-overflow-tooltip="true" prop="createTime" label="创建日期" min-width="20%" />
      <el-table-column min-width="20%" prop="updateTime" label="编辑时间" />
      <el-table-column label="操作" min-width="45%" fixed="right">
        <template slot-scope="scope">
          <el-button type="success" class="filter-item" style="width: 80px;" size="small" @click="saveOrUpdate(scope.row)">编辑分组</el-button>
          <el-button type="danger" class="filter-item" style="width: 80px;" size="small" @click="deleteData(scope.row)">删除分组</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
    >
      <el-form ref="shellDescribe" :model="addData" label-width="100px" class="demo-ruleForm">
        <el-form-item label="分组名称">
          <el-input v-model="addData.shellDescribe" placeholder="请输入分组名称" clearable />
        </el-form-item>
        <el-form-item label="登录用户名">
          <el-input v-model="addData.shellUserName" placeholder="请输入shell登录用户名" clearable />
        </el-form-item>
        <el-form-item label="登录密码">
          <el-input v-model="addData.shellUserPassword" placeholder="请输入shell登录密码" clearable />
        </el-form-item>
        <el-form-item label="执行的命令">
          <el-input v-model="addData.shellCommand" placeholder="请输入执行命令" clearable />
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addData.id == null? add() : update()">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog
      title="提示"
      :visible.sync="del"
      width="30%"
    >
      <span>是否确认删除</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="del = false">取 消</el-button>
        <el-button type="primary" @click="dele()">确 定 删 除</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import request from '@/utils/request'
import qs from 'qs'
export default {
  name: 'Shell',
  data() {
    return {
      list: null,
      total: 0,
      title: null,
      dialogVisible: false,
      del: false,
      addData: {
        id: null,
        shellDescribe: null,
        shellUserName: null,
        shellUserPassword: null,
        shellCommand: null
      },
      listQuery: {
        pageNumber: 1,
        pageSize: 20,
        shellDescribe: null
      }
    }
  },
  created() {
    //  this.listQuery.dateTime = this.getNowFormatDate()
    this.getList()
  },
  methods: {

    saveOrUpdate(row) {
      this.dialogVisible = true
      if (row == null || row.id == null) {
        this.clear()
        this.title = '创建执行分组'
      } else {
        this.title = '修改执行分组'
        this.addData.id = row.id
        this.addData.shellCommand = row.shellCommand
        this.addData.shellDescribe = row.shellDescribe
        this.addData.shellUserName = row.shellUserName
        this.addData.shellUserPassword = row.shellUserPassword
      }
    },
    add() {
      request({
        url:
          '/api/shellBase/addShellBase/',
        method: 'post',
        data: JSON.stringify(this.addData)
      })
        .then(response => {
          this.dialogVisible = false
          this.getList()
          this.clear()
          this.$notify.success({
            title: response.timestamp,
            message: response.msg
          })
        })
        .catch(error => {
          console.log(error)
        })
    },

    update() {
      request({
        url:
          '/api/shellBase/updateShellBase/',
        method: 'post',
        data: JSON.stringify(this.addData)
      })
        .then(response => {
          this.dialogVisible = false
          this.getList()
          this.clear()
          this.$notify.success({
            title: response.timestamp,
            message: response.msg
          })
        })
        .catch(error => {
          console.log(error)
        })
    },
    getList(callback) {
      request({
        url:
          '/api/shellBase/getShellBase?' +
          qs.stringify(this.listQuery, { indices: false }),
        method: 'get'
      })
        .then(response => {
          this.total = response.data.total
          this.list = response.data
        })
        .then(() => {
          if (callback != null) {
            callback()
          }
        })
        .catch(error => {
          console.log(error)
        })
    },
    handleFilter() {
      this.listQuery.pageNumber = 1
      this.getList()
    },
    clear() {
      this.addData.id = null
      this.addData.shellCommand = null
      this.addData.shellDescribe = null
      this.addData.shellUserName = null
      this.addData.shellUserPassword = null
    },
    deleteData(row) {
      console.log(row)
      this.del = true
      this.addData.id = row.id
    },
    dele() {
      request({
        url:
          '/api/shellBase/delShellBase?id=' + this.addData.id,
        method: 'post'
      })
        .then(response => {
          this.clear()
          this.getList()
          this.del = false
          this.$notify.error({
            title: response.timestamp,
            message: response.msg
          })
        })
        .catch(error => {
          console.log(error)
        })
    }
  }

}
</script>

