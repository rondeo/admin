<template>
  <div class="mod-role">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <!-- <el-form-item>
        <el-input v-model="dataForm.roleName" placeholder="活动名称" clearable></el-input>
      </el-form-item> -->
      <el-form-item>
        <!-- <el-button @click="getDataList()">查询</el-button> -->
        <el-button type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <!-- <el-button type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button> -->
        <el-button  type="default" @click="backLastPage()">返回</el-button>
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
        label="地址id">
      </el-table-column>
      <el-table-column
        prop="addressName"
        header-align="center"
        align="center"
        label="地址名称">
      </el-table-column>
      <el-table-column
        prop="remark"
        header-align="center"
        align="center"
        label="备注">
        <!-- <template slot-scope="scope"> -->
          <!-- <el-tag v-if="scope.row.status === 0" size="small" type="danger">禁用</el-tag> -->
          <!-- <el-tag v-else size="small">正常</el-tag> -->正常
        <!-- </template> -->
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="primary" size="small" @click="addOrUpdateHandle(scope.row.roleId)">修改</el-button>
          <el-button type="danger" size="small" @click="deleteHandle(scope.row.roleId)">删除</el-button>
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
    <add-or-update v-if="addOrUpdateVisible" v-bind:activityId="activityId2" v-bind:activityName="activityName2" ref="addOrUpdate" @refreshDataList="getAddressList(activityId2,activityName2)"></add-or-update>
  </div>
</template>

<script>
  import AddOrUpdate from './activity-address-add-or-update'
  export default {
     name: 'page2',
     props: {
      activityId: "",
      activityName:""
    },
    data () {
      return {
        dataForm: {
          roleName: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        activityId2:null,
        activityName2:''
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      // this.getDataList()
    },
    methods: {
      // 获取数据列表
      getAddressList (activityId,activityName) {
        this.$loading.open()
        this.activityId2 = activityId;
        this.activityName2 = activityName;
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/exchange-code/activityAddress/getPageList'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'pageSize': this.pageSize,
            'activityId': activityId
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
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id,this.activityId2,this.activityName2)
        })
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
      backLastPage(){
        this.$emit('refreshDataList')
      }
    }
  }
</script>
