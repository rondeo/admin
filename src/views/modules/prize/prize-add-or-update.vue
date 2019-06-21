<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="奖品名称" prop="prizeName">
        <el-input v-model="dataForm.prizeName" placeholder="奖品名称"></el-input>
      </el-form-item>
      <el-form-item label="奖品code" prop="prizeCode" :class="{ 'is-required': !dataForm.id }">
        <el-input v-model="dataForm.prizeCode" placeholder="奖品code"></el-input>
      </el-form-item>
      <el-form-item label="备注">
        <el-input type="textarea" v-model="dataForm.remark" class="article-textarea" :rows="5">
        </el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import { isEmail, isMobile } from "@/utils/validate";
export default {
  data() {
    return {
      visible: false,
      roleList: [],
      dataForm: {
        id: 0,
        prizeName: "",
        prizeCode: "",
        remark: "",
        status: null
      },
      dataRule: {
        prizeName: [
          { required: true, message: "奖品名称不能为空", trigger: "blur" }
        ],
        prizeCode: [
          { required: true, message: "奖品code不能为空", trigger: "blur" }
        ]
      }
    };
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
      });
      if (this.dataForm.id) {
        this.$http({
          url: this.$http.adornUrl(
            `/exchange-code/prize/getPrize?id=${this.dataForm.id}`
          ),
          method: "get",
          params: this.$http.adornParams()
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.dataForm.prizeName = data.data.prizeName;
            this.dataForm.prizeCode = data.data.prizeCode;
            this.dataForm.status = data.data.status;
            this.dataForm.remark = data.data.remark;
          }
        });
      }
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          if (this.dataForm.id == 0) {
            this.$confirm(`确定进行[新增]操作?`, "提示", {
              confirmButtonText: "确定",
              cancelButtonText: "取消",
              type: "warning"
            }).then(() => {
              this.$loading.open()
              this.$http({
                url: this.$http.adornUrl(`/exchange-code/prize/add`),
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
            });
          } else {
            this.$confirm(`确定进行[编辑]操作?`, "提示", {
              confirmButtonText: "确定",
              cancelButtonText: "取消",
              type: "warning"
            }).then(() => {
              this.$loading.open()
              this.$http({
                url: this.$http.adornUrl(`/exchange-code/prize/update`),
                method: "put",
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
            });
          }
        }
      });
    }
  }
};
</script>
