<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="类型" prop="type">
        <el-radio-group v-model="dataForm.type">
          <el-radio v-for="(type, index) in dataForm.typeList" :label="index" :key="index">{{ type }}</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item :label="dataForm.typeList[dataForm.type] + '名称'" prop="name">
        <el-input v-model="dataForm.name" :placeholder="dataForm.typeList[dataForm.type] + '名称'"></el-input>
      </el-form-item>
      <el-form-item label="上级" prop="parentName">
        <el-popover
          ref="hallListPopover"
          placement="bottom-start"
          trigger="click">
          <el-tree
            :data="hallList"
            :props="hallListTreeProps"
            node-key="hallId"
            ref="hallListTree"
            @current-change="hallListTreeCurrentChangeHandle"
            :default-expand-all="true"
            :highlight-current="true"
            :expand-on-click-node="false">
          </el-tree>
        </el-popover>
        <el-input v-model="dataForm.parentName" v-popover:hallListPopover :readonly="true" placeholder="点击选择上级" class="hall-list__input"></el-input>
      </el-form-item>
      <el-form-item label="排序号" prop="orderNum">
        <el-input-number v-model="dataForm.orderNum" controls-position="right" :min="0" label="排序号"></el-input-number>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import { treeDataTranslate } from '@/utils'
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          type: 1,
          typeList: ['展厅', '展区', '展项'],
          name: '',
          parentId: 0,
          parentName: '',
          orderNum: 0
        },
        dataRule: {
          name: [
            { required: true, message: '名称不能为空', trigger: 'blur' }
          ],
          parentName: [
            { required: true, message: '上级不能为空', trigger: 'change' }
          ]
        },
        hallList: [],
        hallListTreeProps: {
          label: 'name',
          children: 'children'
        }
      }
    },
    created () {
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.$http({
          url: this.$http.adornUrl('/sys/hall/tree'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.hallList = treeDataTranslate(data.hallList, 'hallId')
        }).then(() => {
          this.visible = true
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
          })
        }).then(() => {
          if (!this.dataForm.id) {
            // 新增
            this.hallListTreeSetCurrentNode()
          } else {
            // 修改
            this.$http({
              url: this.$http.adornUrl(`/sys/hall/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              this.dataForm.id = data.hall.hallId
              this.dataForm.type = data.hall.type
              this.dataForm.name = data.hall.name
              this.dataForm.parentId = data.hall.parentId
              this.dataForm.orderNum = data.hall.orderNum
              this.hallListTreeSetCurrentNode()
            })
          }
        })
      },
      // 菜单树选中
      hallListTreeCurrentChangeHandle (data, node) {
        this.dataForm.parentId = data.hallId
        this.dataForm.parentName = data.name
      },
      // 菜单树设置当前选中节点
      hallListTreeSetCurrentNode () {
        this.$refs.hallListTree.setCurrentKey(this.dataForm.parentId)
        this.dataForm.parentName = (this.$refs.hallListTree.getCurrentNode() || {})['name']
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/sys/hall/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'hallId': this.dataForm.id || undefined,
                'type': this.dataForm.type,
                'name': this.dataForm.name,
                'parentId': this.dataForm.parentId,
                'orderNum': this.dataForm.orderNum
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

<style lang="scss">
  .mod-hall {
    .hall-list__input,
    .icon-list__input {
       > .el-input__inner {
        cursor: pointer;
      }
    }
    &__icon-popover {
      max-width: 370px;
    }
    &__icon-list {
      max-height: 300px;
      padding: 0;
      margin: -8px 0 0 -8px;
      > .el-button {
        padding: 8px;
        margin: 8px 0 0 8px;
        > span {
          display: inline-block;
          vertical-align: middle;
          width: 18px;
          height: 18px;
          font-size: 18px;
        }
      }
    }
    .icon-list__tips {
      font-size: 13px;
      text-align: center;
      color: #e6a23c;
      cursor: pointer;
    }
  }
</style>
