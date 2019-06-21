<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="活动">
        <el-input v-model="activityName" :disabled="true"></el-input>
      </el-form-item>
      <el-form-item label="奖品">
       <el-input v-model="prizeName" :disabled="true"></el-input>
      </el-form-item>
      <el-form-item label="奖品库存" prop="prizeNumber">
        <el-input-number size="medium" v-model="dataForm.num" :min="1"></el-input-number>
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
      prizeName:'',
      dataForm: {
        activityId: "",
        prizeId: "",
        num: 1
      },
      prizeList: [],
      dataRule: {
        activityId: [
          { required: true, message: "请选择活动", trigger: "blur" }
        ],
        prizeId: [{ required: true, message: "请选择奖品", trigger: "blur" }],
        num: [
          { required: true, message: "请输入奖品库存", trigger: "blur" }
        ]
      }
    };
  },
  methods: {
    init(activityId, activityName,prizeId,prizeName) {
      this.dataForm.activityId = activityId;
      this.activityName = activityName;
      this.dataForm.prizeId = prizeId;
      this.prizeName = prizeName;

      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
      })
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
              url: this.$http.adornUrl(
                `/exchange-code/exchangeCode/generateCodes`
              ),
              method: "post",
              data: this.$http.adornData(this.dataForm)
            }).then(({ data }) => {
              if (data && data.code === 0) {
                this.$message({
                  message: "操作成功",
                  type: "success",
                  duration: 1500,
                  onClose: () => {
                    this.visible = false;
                    this.$loading.close()
                    this.$emit("refreshDataList");
                  }
                });
              } else {
                this.$message.error(data.message);
                this.$loading.close()
              }
            });
          }).catch(() => {
            this.$loading.close()
          })
        }
      })
    }
  }
};
</script>
