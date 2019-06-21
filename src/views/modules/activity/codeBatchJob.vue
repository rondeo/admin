<template>
  <div class="mod-role" v-if="batchStatus==='batch'">
    <h4>批次详情</h4>
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <!-- <el-form-item>
        <el-input v-model="dataForm.roleName" placeholder="批次code" clearable></el-input>
      </el-form-item> -->
      <el-form-item>
        <!-- <el-button @click="getDataList()">查询</el-button> -->
        <!-- <el-button type="primary">新增</el-button> -->
        <!-- <el-button type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button> -->
        <el-button type="primary" @click="returnLast()">返回</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border fit highlight-current-row
      v-loading="dataListLoading"
      style="width: 100%;">
      <!-- <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column> -->
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        width="80"
        label="批次ID">
      </el-table-column>
      <el-table-column
        prop="batchCode"
        header-align="center"
        align="center"
        label="批次code">
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        label="状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status === 1" size="small" type="primary">正在生成</el-tag>
          <el-tag v-else-if="scope.row.status === 2" size="small" type="danger">生成失败</el-tag>
          <el-tag v-else-if="scope.row.status === 3" size="small" type="success">生成成功</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="primary" size="small" @click="exportBatch(scope.row)">导出码</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <!-- <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update> -->
  </div>
  <div v-else>
    <!-- <codeList v-if="codeListVisible" ref="codeList" @refreshDataList="getDataList"></codeList> -->
  </div>
</template>

<script>
  // import AddOrUpdate from './activity-prize-add-or-update'
  // import codeList from './codeList'

  export default {
    data () {
      return {
        dataForm: {
          roleName: ''
        },
        activityId:null,
        prizeId:null,
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        codeListVisible: false,
        batchStatus: 'batch'
      }
    },
    components: {
      // AddOrUpdate,
      // codeList
    },

    methods: {
      // 获取数据列表
      getBatchList (activityId,prizeId) {
        this.$loading.open()
        this.dataListLoading = true
        this.activityId = activityId
        this.prizeId = prizeId
        this.batchStatus = "batch";
        this.$http({
          url: this.$http.adornUrl('/exchange-code/batch/getBatchPageList'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'pageSize': this.pageSize,
            'activityId': activityId,
            'prizeId': prizeId
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.data.list
            this.totalPage = data.data.total
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
          this.$loading.close()
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      getCodeList(id){
        this.codeListVisible = true
        this.batchStatus = "code";
        this.$nextTick(() => {
          this.$refs.codeList.getDataList()
        })
      },
      //导出码
      exportBatch(row){
        debugger
        this.$http({
          url: this.$http.adornUrl('/exchange-code/exchangeCode/getCodes'),
          method: 'get',
          params: this.$http.adornParams({
            'batchId': row.id
          })
        }).then(response => {
          if (response.data.data.length > 0) {
            import("@/vendor/Export2Excel").then(excel => {
              const tHeader = [
                  "兑换码",
                  "所属活动",
                  "兑换奖品"
              ];
              const filterVal = [
                  "exchangeCode",
                  "activityName",
                  "prizeName"
              ];
              let list = [];
              list = response.data.data;
              const data = this.formatJson(filterVal, list);
              let fileName = response.data.data[0].activityName+"_"+row.batchCode+"_"+row.id
              excel.export_json_to_excel({
                  header: tHeader,
                  data,
                  filename: fileName,
                  autoWidth: true
              });
            });
          }
        })
      },
      formatJson(filterVal, jsonData) {
        return jsonData.map(v =>
            filterVal.map(j => {
              return v[j];
            })
        );
    },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.roleId
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/exchange-code/sys/role/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }).catch(() => {})
      },
      returnLast(){
        this.$emit('refreshDataList')
      }
    }
  }
</script>
