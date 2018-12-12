<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="终端名称" prop="pcName">
      <el-input v-model="dataForm.pcName" placeholder="终端名称"></el-input>
    </el-form-item>
    <el-form-item label="IP" prop="ip">
      <el-input v-model="dataForm.ip" placeholder="IP"></el-input>
    </el-form-item>
    <el-form-item label="MAC" prop="mac">
      <el-input v-model="dataForm.mac" placeholder="MAC"></el-input>
    </el-form-item>
    <el-form-item label="默认媒体" prop="defaultContent">
      <el-input v-model="dataForm.defaultContent" placeholder="默认媒体内容"></el-input>
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
          pcName: '',
          ip: '',
          mac: '',
          defaultContent: '',
          hallA: '',
          hallB: '',
          hallC: '',
          operator: '',
          operateTime: ''
        },
        dataRule: {
          pcName: [
            { required: true, message: '终端名称不能为空', trigger: 'blur' }
          ],
          ip: [
            { required: true, message: 'IP不能为空', trigger: 'blur' }
          ],
          mac: [
            { required: true, message: 'MAC不能为空', trigger: 'blur' }
          ],
          defaultContent: [
            { required: true, message: '默认媒体内容不能为空', trigger: 'blur' }
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
      init (id, node) {
        if (node.type === 0) {
          this.dataForm.hallA = node.hallId
        }
        if (node.type === 1) {
          this.dataForm.hallA = node.parentId
          this.dataForm.hallB = node.hallId
        }
        if (node.type === 2) {
          this.dataForm.hallB = node.parentId
          this.dataForm.hallC = node.hallId
          this.$http({
            url: this.$http.adornUrl(`/sys/hall/info/${node.parentId}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            this.dataForm.hallA = data.hall.parentId
          })
        }
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/sys/terminal/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.pcName = data.terminal.pcName
                this.dataForm.ip = data.terminal.ip
                this.dataForm.mac = data.terminal.mac
                this.dataForm.defaultContent = data.terminal.defaultContent
                this.dataForm.operator = data.terminal.operator
                this.dataForm.operateTime = data.terminal.operateTime
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
              url: this.$http.adornUrl(`/sys/terminal/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'pcName': this.dataForm.pcName,
                'ip': this.dataForm.ip,
                'mac': this.dataForm.mac,
                'defaultContent': this.dataForm.defaultContent,
                'hallA': this.dataForm.hallA,
                'hallB': this.dataForm.hallB,
                'hallC': this.dataForm.hallC,
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
