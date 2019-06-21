<template>
  <div class="page1">
    <el-form :model="dataForm" style="width:60%" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="活动名称" prop="activityName">
        <el-input v-model="dataForm.activityName" placeholder="活动名称"></el-input>
      </el-form-item>
      <el-form-item label="活动code" prop="activityCode">
        <el-input v-model="dataForm.activityCode" :disabled="true" placeholder="活动code"></el-input>
      </el-form-item>
      <el-form-item label="活动时间" required>
        <el-col :span="11">
          <el-form-item prop="activityTimeStart">
            <el-date-picker :disabled="isDisabled"  placeholder="选择日期" v-model="dataForm.activityTimeStart" style="width: 100%;"></el-date-picker>
          </el-form-item>
        </el-col>
        <el-col class="line" :span="2" style="text-align:center">至</el-col>
        <el-col :span="11">
          <el-form-item prop="activityTimeEnd">
            <el-date-picker :disabled="isDisabled" placeholder="选择日期" v-model="dataForm.activityTimeEnd" style="width: 100%;"></el-date-picker>
          </el-form-item>
        </el-col>
      </el-form-item>
      <el-form-item label="活动状态" prop="state" v-if="dataForm.status == 2">
        <el-radio-group v-model="dataForm.state" >
          <el-radio :label="1" :disabled="true">不公开</el-radio>
          <el-radio :label="2"  :disabled="true">公开</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="活动状态" prop="state" v-else>
        <el-radio-group v-model="dataForm.state" >
          <el-radio :label="1">不公开</el-radio>
          <el-radio :label="2">公开</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="备注" prop="remark">
        <el-input  type="textarea" v-model="dataForm.remark" class="article-textarea" :rows="5"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button  type="default" @click="backLastPage()">返回</el-button>
        <el-button  type="primary" @click="dataFormSubmit()">保存</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
  // import AddOrUpdate from './activity-address-add-or-update'
  import { formatDate } from '@/utils'
  export default {
    props: {
      activityId: ""
    },
    data () {
      return {
        listLoading:false,
        isDisabled:false,
        dataForm: {
          id: 0,
          activityName: '',
          activityCode:'',
          activityTimeStart:'',
          activityTimeEnd:'',
          status:1,
          state:1,
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
            {required: true, message: '请选择活动开始时间', trigger: 'blur' }
          ],
          activityTimeEnd: [
            {required: true, message: '请选择活动结束时间', trigger: 'blur' }
          ],
          state: [
            { required: true, message: '请选择活动状态', trigger: 'change' }
          ]
        },
      }
    },
    components: {
      // AddOrUpdate
    },
    activated () {
      // this.getDataList()
    },
    filters: {
      formatDate(time) {
        var date = new Date(time)
        return formatDate(date, 'yyyy-MM-dd')
      }
    },
    methods: {
      init (id) {
        this.$loading.open()
        this.dataForm.id = id || 0
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/exchange-code/activity/getActivityDetail?id=${this.dataForm.id}`),
            method: 'get',
            // params: this.$http.adornParams()
          }).then(response => {
            let data = response.data
            if (data && data.code === 0) {
              this.dataForm.activityName = data.data.activityName;
              this.dataForm.activityCode =data.data.activityCode;
              this.dataForm.activityTimeStart = this.formatDate(data.data.activityTimeStart);
              this.dataForm.activityTimeEnd = this.formatDate(data.data.activityTimeEnd);
              this.dataForm.status = data.data.status;
              this.dataForm.state = data.data.status;
              this.dataForm.remark = data.data.remark;
              if(data.data.status == 2){
                this.isDisabled = true;
              }
            }
            this.$loading.close()
          })
        }
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$confirm(`确定进行[编辑]操作?`, '提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning'
            }).then(() => {
              this.dataForm.status = this.dataForm.state
              this.$loading.open()
              this.$http({
                url: this.$http.adornUrl(`/exchange-code/activity/updateActivity`),
                method: 'post',
                data: this.dataForm
              }).then(({data}) => {
                if (data && data.code === 0) {
                  this.$message({
                    message: '操作成功',
                    type: 'success',
                    duration: 1500,
                    onClose: () => {
                       this.$loading.close()
                      this.$emit('refreshDataList')
                    }
                  })

                } else {
                  this.$message.error(data.message)
                  this.$loading.close()
                }
              })
            })
          }
        })
      },
      formatDate(time) {
        var date = new Date(time)
        return formatDate(date, 'yyyy-MM-dd')
      },
      backLastPage(){
        this.$emit('refreshDataList')
      }
    }
  }
</script>
