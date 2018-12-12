<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="标题" prop="title">
      <el-input v-model="dataForm.title" placeholder="标题"></el-input>
    </el-form-item>
    <el-form-item label="参数" prop="param">
      <el-input v-model="dataForm.param" placeholder="参数"></el-input>
    </el-form-item>
    <el-form-item label="区域" prop="position">
      <el-input v-model="dataForm.position" placeholder="区域"></el-input>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          title: '',
          type: '',
          suffix: '',
          path: '',
          fileName: '',
          fileSize: '',
          state: '',
          param: '',
          position: '',
          operator: '',
          operateTime: ''
        },
        dataRule: {
          title: [
            { required: true, message: '标题不能为空', trigger: 'blur' }
          ],
          type: [
            { required: true, message: '媒体类型不能为空', trigger: 'blur' }
          ],
          suffix: [
            { required: true, message: '文件后缀不能为空', trigger: 'blur' }
          ],
          path: [
            { required: true, message: '存储路径不能为空', trigger: 'blur' }
          ],
          fileName: [
            { required: true, message: '文件本地名称不能为空', trigger: 'blur' }
          ],
          fileSize: [
            { required: true, message: '文件大小不能为空', trigger: 'blur' }
          ],
          state: [
            { required: true, message: '状态不能为空', trigger: 'blur' }
          ],
          param: [
            { required: true, message: '参数不能为空', trigger: 'blur' }
          ],
          position: [
            { required: true, message: '区域不能为空', trigger: 'blur' }
          ],
          operator: [
            { required: true, message: '操作人不能为空', trigger: 'blur' }
          ],
          operateTime: [
            { required: true, message: '操作时间不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/sys/content/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.title = data.content.title
                this.dataForm.type = data.content.type
                this.dataForm.suffix = data.content.suffix
                this.dataForm.path = data.content.path
                this.dataForm.fileName = data.content.fileName
                this.dataForm.fileSize = data.content.fileSize
                this.dataForm.state = data.content.state
                this.dataForm.param = data.content.param
                this.dataForm.position = data.content.position
                this.dataForm.operator = data.content.operator
                this.dataForm.operateTime = data.content.operateTime
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/sys/content/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'title': this.dataForm.title,
                'type': this.dataForm.type,
                'suffix': this.dataForm.suffix,
                'path': this.dataForm.path,
                'fileName': this.dataForm.fileName,
                'fileSize': this.dataForm.fileSize,
                'state': this.dataForm.state,
                'param': this.dataForm.param,
                'position': this.dataForm.position,
                'operator': this.dataForm.operator,
                'operateTime': this.dataForm.operateTime
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
