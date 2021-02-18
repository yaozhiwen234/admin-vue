<template>
  <div class="app-container">

    <div style="text-align: center;">
      <el-row>
        <el-button v-if="view=='compile'" icon="el-icon-upload" class="filter-item" size="small" type="warning" align="center" style="float: left;width: 120px;" @click="centerDialogVisible=true">点击文件上传</el-button>
        <h4 style="display:inline;">您正在 {{ view =='preview'?'预览':'编辑' }}  {{ listQuery.title }} 文本</h4>
        <el-button v-if="view=='compile'" class="filter-item" size="small" style="float: right;width: 80px;" align="center" type="primary" @click="add('add')">确定</el-button>
        <el-button v-if="view=='compile'" class="filter-item" size="small" style="float: right;width: 80px;" align="center" type="warning" @click="hreftwo()">取消</el-button>
        <el-button v-if="view=='preview'" class="filter-item" size="small" style="float: right;width: 80px;" align="center" type="warning" @click="hreftwo()">退出</el-button>
      </el-row>
    </div>
    <el-row>
      <mavon-editor
        v-if="view=='preview'"
        ref="md"
        v-model="listQuery.text"
        :ishljs="true"
        :subfield="false"
        :toolbars-flag="prop.toolbarsFlag"
        :editable="prop.editable"
        :scroll-style="prop.scrollStyle"
        :default-open="prop.defaultOpen"
        :style="'height:' + height"
      />
      <mavon-editor
        v-else
        ref="md"
        v-model="listQuery.text"
        :style="'height:' + height"
        @save="add('ctrls')"
      />
    </el-row>
    <el-dialog
      title="上传MarkDown文件"
      :visible.sync="centerDialogVisible"
      width="40%"
      center
    >
      <el-upload
        class="upload-demo"
        action="https://jsonplaceholder.typicode.com/posts/"
        accept=".md"
        :before-upload="handleBeforeUpload"
        drag
      >
        <i class="el-icon-upload" />
        <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
      </el-upload>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" :disabled="name.length!=0?fale:true" style="width: 80px;" @click="fileUpload()">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
import { upload } from '@/utils/upload'
import { mapGetters } from 'vuex'
import request from '@/utils/request'
import qs from 'qs'

export default {
  name: 'Markdown',
  data() {
    return {
      centerDialogVisible: false,
      name: '',
      data: '',
      listQuery: {
        id: '',
        text: '',
        title: '',
        operation: ''
      },
      view: '',
      height: document.documentElement.clientHeight - 200 + 'px',
      listener: ''
    }
  },
  computed: {
    ...mapGetters([
      'imagesUploadApi',
      'baseApi'
    ]),
    prop() {
      const data = {
        subfield: false, // 单双栏模式
        defaultOpen: 'preview', // edit： 默认展示编辑区域 ， preview： 默认展示预览区域
        editable: false,
        toolbarsFlag: false,
        scrollStyle: true
      }
      return data
    }
  },
  mounted() {
    const that = this
    window.onresize = function temp() {
      that.height = document.documentElement.clientHeight - 200 + 'px'
    }
  },
  created() {
    this.listQuery.id = this.$route.query.rows.id
    this.listQuery.text = this.$route.query.rows.text
    this.listener = this.$route.query.rows.text
    this.listQuery.title = this.$route.query.rows.title
    this.view = this.$route.query.view
  },
  methods: {
    handleBeforeUpload(file) {
      const fileExt = file.name.split('.').pop().toLocaleLowerCase()
      if (fileExt === 'md') {
        this.file = file
        this.readData(file)
      } else {
        this.$notify.error({
          message: '请选择后缀为"md"的文件'
        })
      }
      // return false
    },
    readData(file) {
      const fileName = file.name
      const reader = new FileReader()
      reader.readAsText(file)
      if (typeof FileReader === 'undefined') {
        this.$message({
          type: 'info',
          message: '您的浏览器不支持FileReader接口'
        })
        return false
      }
      reader.onerror = e => {
        this.$notify.error({
          message: '读取文件错误'
        })
        return false
      }
      reader.onload = e => {
        this.name = fileName
        this.data = e.target.result
      }
    },
    fileUpload() {
      if (this.listQuery.text === '' || this.listQuery.text.length === 0) {
        this.listQuery.text = this.data
        this.centerDialogVisible = false
      } else {
        this.fileOpen()
      }
    },
    fileOpen() {
      this.$confirm('您是合并当前内容还是覆盖当前内容', '提示', {
        distinguishCancelAndClose: true,
        confirmButtonText: '合并',
        cancelButtonText: '覆盖',
        type: 'warning'
      }).then(() => {
        this.listQuery.text = this.listQuery.text + '\r\n' + this.data
        this.centerDialogVisible = false
        this.$message({
          type: 'success',
          message: '合并成功!'
        })
      }).catch(action => {
        if (action === 'cancel') {
          this.listQuery.text = this.data
          this.centerDialogVisible = false
          this.$message({
            type: 'success',
            message: '覆盖成功'
          })
        } else {
          this.centerDialogVisible = false
        }
      })
    },
    imgAdd(pos, $file) {
      upload(this.imagesUploadApi, $file).then(res => {
        const data = res.data
        const url = this.baseApi + '/file/' + data.type + '/' + data.realName
        this.$refs.md.$img2Url(pos, url)
      })
    },
    open() {
      this.$confirm('此操作没有永久保存该文件, 是否永久保存', '提示', {
        confirmButtonText: '永久保存',
        cancelButtonText: '退出',
        type: 'warning'
      }).then(() => {
        this.add('add')
      }).catch(() => {
        this.$router.go(-1)
        this.$message({
          type: 'error',
          message: '文件未保存'
        })
      })
    },
    hreftwo() {
      if (this.listener === this.listQuery.text) {
        this.$router.go(-1)
      } else {
        this.open()
      }
    },
    add(operation) {
      this.listQuery.operation = operation
      request({
        url:
          '/api/article/addArticle?' +
            qs.stringify(this.listQuery, { indices: false }),
        method: 'put'
      })
        .then(response => {
          if (response.code === 200) {
            if (operation === 'add') {
              this.$router.go(-1)
              this.$notify.error({
                title: response.timestamp,
                message: response.msg
              })
            }
          }
        })
        .catch(error => {
          console.log(error)
        })
    }
  }

}
</script>

<style>
.el-dialog .el-dialog__body{
      display: flex;
      justify-content: center;
      align-items: center;
}
</style>
