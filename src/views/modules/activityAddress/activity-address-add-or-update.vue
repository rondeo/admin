<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="活动" prop="activityId">
        <el-select clearable class="filter-item" style="width: 100%" v-model="dataForm.activityId" placeholder="请选择活动">
          <!-- <el-option v-for="room in activityList" :key="room.id" :label="room.roomName" :value="room.id">
          </el-option> -->
          <el-option label="活动一" value="1"></el-option>
          <el-option label="活动二" value="2"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="活动地址" prop="addressName">
        <el-input v-model="dataForm.addressName" placeholder="活动code"></el-input>
      </el-form-item>
      <el-form-item label="备注" prop="remark">
        <el-input  type="textarea" v-model="dataForm.remark" class="article-textarea" :rows="5"></el-input>
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
        menuList: [],
        menuListTreeProps: {
          label: 'name',
          children: 'children'
        },
        dataForm: {
          id: 0,
          activityId: '',
          addressName:'',
          remark: ''
        },
        dataRule: {
           activityId: [
            { required: true, message: '请选择活动', trigger: 'blur' }
          ],
           addressName: [
            { required: true, message: '活动地址不能为空', trigger: 'blur' }
          ]
        },
        tempKey: -666666 // 临时key, 用于解决tree半选中状态项不能传给后台接口问题. # 待优化
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0

          this.visible = true
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
          })
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/exchange-code/sys/role/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.activityId = data.role.activityId
                this.dataForm.addressName = data.role.addressName
                this.dataForm.remark = data.role.remark
              }
            })
          }
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/exchange-code/sys/role/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData(this.dataForm)
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
