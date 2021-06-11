<template>
  <div class="app-container">
    <!--工具栏-->
    <el-input v-model="listQuery.title" size="small" style="width: 150px;" prefix-icon="el-icon-search" class="filter-item" placeholder="请输入标题" clearable />
    <el-select v-model="listQuery.tags" size="small" style="width: 150px;" clearable filterable placeholder="请选择标签">
      <el-option
        v-for="item in tagsList"
        :key="item"
        :label="item"
        :value="item"
      />
    </el-select>
    <el-select v-model="listQuery.categories" size="small" style="width: 150px;" clearable filterable placeholder="请选择分类">
      <el-option
        v-for="item in categorieList"
        :key="item"
        :label="item"
        :value="item"
      />
    </el-select>

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

    <el-button size="small" class="filter-item" style="width: 80px;" type="success" @click="handleFilter()">查询</el-button>
    <el-button size="small" class="filter-item" style="width: 80px;" :disabled="depl===true?false:true" type="warning" @click="deploy()">发布</el-button>
    <el-button size="small" class="filter-item" style="width: 80px;" type="primary" @click="creatTitle()">新建文章</el-button>
    <el-button size="small" class="filter-item" style="width: 80px;" :disabled="multipleSelection.length===1?false:true" type="warning" @click="updateTitle()">修改标题</el-button>
    <el-button size="small" class="filter-item" style="width: 80px;" type="danger" :disabled="multipleSelection.length===0?true:false" @click="del=true">删除</el-button>
    <el-button size="small" class="filter-item" style="width: 80px;" type="danger" @click="fileDB=true">导入数据</el-button>
    <el-table v-loading="listLoading" :data="list" style="width: 100%;" @selection-change="handleSelectionChange">
      <el-table-column type="selection" min-width="1%" />
      <el-table-column :show-overflow-tooltip="true" prop="title" label="标题" min-width="15%" />
      <el-table-column :show-overflow-tooltip="true" prop="tags" label="标签" min-width="15%" />
      <el-table-column :show-overflow-tooltip="true" prop="categories" label="分类" min-width="15%" />
      <el-table-column :show-overflow-tooltip="true" prop="cover" min-width="20%" label="图片链接" />
      <el-table-column :show-overflow-tooltip="true" prop="date" label="创建文件日期" min-width="20%" />
      <el-table-column min-width="20%" prop="createTime" label="文本编辑时间" />
      <el-table-column label="操作" min-width="45%" fixed="right">
        <template slot-scope="scope">
          <el-button type="primary" class="filter-item" style="width: 80px;" size="small" @click="hreftwo(scope.row,'compile')">编辑文本</el-button>
          <el-button type="success" class="filter-item" style="width: 80px;" size="small" @click="hreftwo(scope.row,'preview')">预览文本</el-button>
          <el-button type="info" class="filter-item" style="width: 80px;" size="small" :disabled="scope.row.state===1?true:false" @click="synText(scope.row)">同步数据</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog
      title="新建一个文件"
      :visible.sync="dialogVisible"
      width="30%"
    >
      <el-form ref="addArticleHead" :model="addArticleHead" status-icon :rules="rules" label-width="100px" class="demo-ruleForm">
        <el-form-item label="标题" prop="title">
          <el-input v-model="addArticleHead.title" placeholder="请输入标题" clearable />
        </el-form-item>
        <el-form-item label="标签">
          <el-input v-model="addArticleHead.tags" placeholder="请输入标签" clearable />
        </el-form-item>
        <el-form-item label="分类">
          <el-input v-model="addArticleHead.categories" placeholder="请输入分类" clearable />
        </el-form-item>
        <el-form-item label="主题图片连接">
          <el-input v-model="addArticleHead.cover" placeholder="请输入图片链接" clearable />
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addHead('addArticleHead')">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog
      title="修改文件标题"
      :visible.sync="updateVisible"
      width="30%"
    >
      <el-form ref="addArticleHead" :model="addArticleHead" status-icon label-width="100px" class="demo-ruleForm">
        <el-form-item
          label="标题"
          prop="title"
          :rules="[
            { required: true, message: '请输入标题', trigger: 'blur' }

          ]"
        >
          <el-input v-model="addArticleHead.title" placeholder="请输入标题" clearable />
        </el-form-item>
        <el-form-item label="标签">
          <el-input v-model="addArticleHead.tags" placeholder="请输入标签" clearable />
        </el-form-item>
        <el-form-item label="分类">
          <el-input v-model="addArticleHead.categories" placeholder="请输入分类" clearable />
        </el-form-item>
        <el-form-item label="主题图片连接">
          <el-input v-model="addArticleHead.cover" placeholder="请输入图片链接" clearable />
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="updateVisible = false">取 消</el-button>
        <el-button type="primary" @click="update()">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog
      title="导入数据"
      :visible.sync="fileDB"
      width="30%"
    >
      文件路径： <el-input v-model="filePath" />
      <span slot="footer" class="dialog-footer">
        <el-button @click="fileDB = false">取 消</el-button>
        <el-button type="primary" :disabled="filePath.length!==0?false:true" @click="addfileDB()">确 定</el-button>
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

    <!--分页组件-->
    <div class="grid-content bg-purple" style="margin-top: -30px;">
      <pagination
        v-show="total > 0"
        :total="total"
        :page.sync="listQuery.pageNumber"
        :limit.sync="listQuery.pageSize"
        @pagination="getList"
      />
    </div>
  </div>

</template>

<script>
import '@riophae/vue-treeselect/dist/vue-treeselect.css'
import DateRangePicker from '@/components/DateRangePicker'
import Pagination from '@/components/Pagination' // secondary package based on el-pagination
import request from '@/utils/request'
import qs from 'qs'
export default {
  name: 'Hexo',
  components: { DateRangePicker, Pagination },
  data() {
    var validateTitle = (rule, value, callback) => {
      if (this.addArticleHead == null) {
        return callback()
      }
      request({
        url:
          '/api/article/showArticleByTitle?title=' + value,
        method: 'get'
      })
        .then(response => {
          if (response.code !== 200) {
            callback(new Error(response.msg))
          } else {
            callback()
          }
        })
        .catch(error => {
          console.log(error)
        })
    }
    return {
      tagsList: '',
      categorieList: '',
      fileDB: false,
      dialogVisible: false,
      addArticleHead: {
        title: null,
        tags: null,
        categories: null,
        cover: null
      },
      depl: true,
      filePath: '',
      listQuery: {
        pageNumber: 1,
        pageSize: 20,
        title: null,
        tags: null,
        categories: null,
        startTime: null,
        endTime: null
      },
      list: null,
      total: 0,
      createTime: null,
      listLoading: false,
      updateVisible: false,
      multipleSelection: [],
      del: false,
      date: [],
      rules: {
        title: [
          { required: true, message: '请输入标题名称', trigger: 'blur' },
          { validator: validateTitle, trigger: 'blur' }
        ]
      }

    }
  },
  created() {
  //  this.listQuery.dateTime = this.getNowFormatDate()
    this.getList()
  },
  methods: {
    getList(callback) {
      this.getCategorie()
      this.getTags()
      this.listLoading = true
      request({
        url:
          '/api/article/showArticle?' +
          qs.stringify(this.listQuery, { indices: false }),
        method: 'get'
      })
        .then(response => {
          this.total = response.data.total
          this.list = response.data.record
          this.listLoading = false
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
    getCategorie() {
      request({
        url:
          '/api/article/showCategorie',
        method: 'get'
      })
        .then(response => {
          this.categorieList = response.data
        })
        .catch(error => {
          console.log(error)
        })
    },
    getTags() {
      request({
        url:
          '/api/article/showTags',
        method: 'get'
      })
        .then(response => {
          this.tagsList = response.data
        })
        .catch(error => {
          console.log(error)
        })
    },
    creatTitle() {
      this.addArticleHead.title = null
      this.addArticleHead.tags = null
      this.addArticleHead.categories = null
      this.addArticleHead.cover = null
      this.dialogVisible = true
    },
    // 创建文件头部
    addHead(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          request({
            url:
          '/api/article/addTitle/',
            method: 'post',
            data: JSON.stringify(this.addArticleHead)
          })
            .then(response => {
              this.getList()
              this.dialogVisible = false

              this.$notify.error({
                title: response.timestamp,
                message: response.msg
              })
            })
            .catch(error => {
              console.log(error)
            })
        } else {
          return false
        }
      })
    },
    handleSelectionChange(val) {
      this.multipleSelection = val
    },
    handleFilter() {
      this.listQuery.pageNumber = 1
      if (this.date && this.date.length === 2) {
        this.listQuery.startTime = this.date[0]
        this.listQuery.endTime = this.date[1]
      } else {
        this.listQuery.startTime = ''
        this.listQuery.endTime = ''
        this.date = []
      }
      this.getList()
    },
    updateTitle() {
      this.addArticleHead.title = this.multipleSelection[0].title
      this.addArticleHead.tags = this.multipleSelection[0].tags
      this.addArticleHead.categories = this.multipleSelection[0].categories
      this.addArticleHead.cover = this.multipleSelection[0].cover

      // this.addArticleHead = this.multipleSelection[0]
      this.updateVisible = true
    },
    dele() {
      var arr = this.multipleSelection
      var id = []
      for (var i = 0; i < arr.length; i++) {
        id[i] = arr[i].id
      }
      request({
        url:
          '/api/article/delArticle?id=' + id,
        method: 'delete'
      })
        .then(response => {
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
    },

    update() {
      console.log(JSON.stringify(this.addArticleHead))
      request({
        url:
          '/api/article/updateArticleByTitle/' + this.multipleSelection[0].id,
        method: 'put',
        data: JSON.stringify(this.addArticleHead)
      })
        .then(response => {
          this.getList()
          this.updateVisible = false
          this.$notify.error({
            title: response.timestamp,
            message: response.msg
          })
        })
        .catch(error => {
          console.log(error)
        })
    },
    getText(row) {
      request({
        url:
          '/api/article/showTest?id=' + row.id + '&title=' + row.title,
        method: 'get'
      })
        .then(response => {
          return response.data.text
        })
        .catch(error => {
          console.log(error)
        })
    },
    hreftwo(row, view) {
      row.text = this.getText(row)
      this.$router.push(
        { path: '/hexo/module/markDown',
          query: {
            rows: row,
            view: view
          }
        })
    },
    synText(row) {
      request({
        url:
          '/api/article/synArticle?id=' + row.id,
        method: 'post'
      })
        .then(response => {
          this.getList()
          this.$notify.error({
            title: response.timestamp,
            message: response.msg
          })
        })
        .catch(error => {
          console.log(error)
        })
    },
    deploy() {
      this.depl = false
      this.listLoading = true
      request({
        url:
          '/api/article/deployArticle',
        method: 'post'
      })
        .then(response => {
          this.getList()
          this.depl = true
          this.$notify.error({
            title: response.timestamp,
            message: response.msg
          })
        })
        .catch(error => {
          console.log(error)
        })
    },
    addfileDB() {
      request({
        url:
          '/api/article/fileWriteDb?filePath=' + this.filePath,
        method: 'post'
      })
        .then(response => {
          this.fileDB = false
          this.filePath
          this.getList()
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

<style rel="stylesheet/scss" lang="scss" scoped>
 ::v-deep .el-input-number .el-input__inner {
    text-align: left;
  }
</style>
