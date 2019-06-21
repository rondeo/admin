<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="活动" prop="activityId">
        <el-input v-model="activityName" :disabled="true"></el-input>
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
export default {
  data() {
    return {
      visible: false,
      activityName: "",
      dataForm: {
        id: 0,
        activityId: "",
        addressName: "",
        remark: ""
      },
      dataRule: {
        activityId: [
          { required: true, message: "请选择活动", trigger: "blur" }
        ],
        addressName: [
          { required: true, message: "活动地址不能为空", trigger: "blur" }
        ]
      },
    };
  },
  methods: {
    init(id, activityId, activityName) {
      this.dataForm.id = id || 0;
      this.dataForm.activityId = activityId;
      this.activityName = activityName;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
      });
      if (this.dataForm.id) {
        this.$http({
          url: this.$http.adornUrl(
            `/exchange-code/sys/role/info/${this.dataForm.id}`
          ),
          method: "get",
          params: this.$http.adornParams()
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.dataForm.activityId = data.role.activityId;
            this.dataForm.addressName = data.role.addressName;
            this.dataForm.remark = data.role.remark;
          }
        });
      }
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.$confirm(`确定进行[新增]操作?`, "提示", {
            confirmButtonText: "确定",
            cancelButtonText: "取消",
            type: "warning"
          }).then(() => {
            this.$loading.open()
            this.$http({
              url: this.$http.adornUrl(`/exchange-code/activityAddress/addAddress`),
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
                    this.$loading.close()
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.message)
                this.$loading.close()
              }
            })
          });
        }
      });
    }
  }
};
</script>
