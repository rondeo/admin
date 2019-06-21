<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="活动名称" prop="activityName">
        <el-input v-model="dataForm.activityName" placeholder="活动名称"></el-input>
      </el-form-item>
      <el-form-item label="活动code" prop="activityCode">
        <el-input v-model="dataForm.activityCode" placeholder="活动code"></el-input>
      </el-form-item>
      <el-form-item label="活动时间" required>
        <el-col :span="11">
          <el-form-item prop="activityTimeStart">
            <el-date-picker type="date" placeholder="选择日期" v-model="dataForm.activityTimeStart " style="width: 100%;"></el-date-picker>
          </el-form-item>
        </el-col>
        <el-col class="line" :span="2" style="text-align:center">至</el-col>
        <el-col :span="11">
          <el-form-item prop="activityTimeEnd">
            <el-date-picker type="date" placeholder="选择日期" v-model="dataForm.activityTimeEnd" style="width: 100%;"></el-date-picker>
          </el-form-item>
        </el-col>
      </el-form-item>
      <el-form-item label="活动状态" prop="status">
        <el-radio-group v-model="dataForm.status">
          <el-radio :label="1">不公开</el-radio>
          <el-radio :label="2">公开</el-radio>
        </el-radio-group>
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
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          activityName: '',
          activityCode:'',
          activityTimeStart:'',
          activityTimeEnd:'',
          status:1,
          remark: ''
        },
        dataRule: {
           activityName: [
            { required: true, message: '活动名称不能为空', trigger: 'blur' }
          ],
           activityCode: [
            { required: true, message: '活动Code不能为空', trigger: 'blur' }
          ],
          activityTimeStart: [
            { type: 'date', required: true, message: '请选择活动开始时间', trigger: 'blur' }
          ],
          activityTimeEnd: [
            { type: 'date', required: true, message: '请选择活动结束时间', trigger: 'blur' }
          ],
          status: [
            { required: true, message: '请选择活动状态', trigger: 'change' }
          ],
        },
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            if (this.dataForm.id == 0) {
              this.$confirm(`确定进行[新增]操作?`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
              }).then(() => {
                this.$loading.open()
                this.$http({
                  url: this.$http.adornUrl(`/exchange-code/activity/addActivity`),
                  method: 'post',
                  data: this.dataForm
                }).then(response => {
                  if (response.data && response.data.code === 0) {
                    this.$message({
                      message: '操作成功',
                      type: 'success',
                      duration: 1500,
                      onClose: () => {
                        this.visible = false
                        this.$loading.close()
                        this.$emit('refreshDataList')
                      }
                    })
                  } else {
                    this.$message.error(response.data.message)
                    this.$loading.close()
                  }
                })
              })
            }

          }
        })
      }
    }
  }
</script>
