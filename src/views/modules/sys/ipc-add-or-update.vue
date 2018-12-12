<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="指令名称" prop="name">
      <el-input v-model="dataForm.name" placeholder="指令名称"></el-input>
    </el-form-item>
    <el-form-item label="指令编码" prop="ipcCode">
      <el-input v-model="dataForm.ipcCode" placeholder="指令编码"></el-input>
    </el-form-item>
    <el-form-item label="指令参数" prop="ipcValue">
      <el-input v-model="dataForm.ipcValue" placeholder="指令参数"></el-input>
    </el-form-item>
    <el-form-item label="展厅" prop="hallA">
      <el-input v-model="dataForm.hallA" placeholder="展厅"></el-input>
    </el-form-item>
    <el-form-item label="展区" prop="hallB">
      <el-input v-model="dataForm.hallB" placeholder="展区"></el-input>
    </el-form-item>
    <el-form-item label="展项" prop="hallC">
      <el-input v-model="dataForm.hallC" placeholder="展项"></el-input>
    </el-form-item>
    <el-form-item label="操作人" prop="operator">
      <el-input v-model="dataForm.operator" placeholder="操作人"></el-input>
    </el-form-item>
    <el-form-item label="操作时间" prop="operateTime">
      <el-input v-model="dataForm.operateTime" placeholder="操作时间"></el-input>
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
          name: '',
          ipcCode: '',
          ipcValue: '',
          hallA: '',
          hallB: '',
          hallC: '',
          operator: '',
          operateTime: ''
        },
        dataRule: {
          name: [
            { required: true, message: '指令名称不能为空', trigger: 'blur' }
          ],
          ipcCode: [
            { required: true, message: '指令编码不能为空', trigger: 'blur' }
          ],
          ipcValue: [
            { required: true, message: '指令参数不能为空', trigger: 'blur' }
          ],
          hallA: [
            { required: true, message: '展厅不能为空', trigger: 'blur' }
          ],
          hallB: [
            { required: true, message: '展区不能为空', trigger: 'blur' }
          ],
          hallC: [
            { required: true, message: '展项不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/sys/ipc/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.ipc.name
                this.dataForm.ipcCode = data.ipc.ipcCode
                this.dataForm.ipcValue = data.ipc.ipcValue
                this.dataForm.hallA = data.ipc.hallA
                this.dataForm.hallB = data.ipc.hallB
                this.dataForm.hallC = data.ipc.hallC
                this.dataForm.operator = data.ipc.operator
                this.dataForm.operateTime = data.ipc.operateTime
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
              url: this.$http.adornUrl(`/sys/ipc/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'ipcCode': this.dataForm.ipcCode,
                'ipcValue': this.dataForm.ipcValue,
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
