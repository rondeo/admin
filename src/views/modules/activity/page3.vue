<template>
  <div class="mod-role" v-if="pageStatus==='list'">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <!-- <el-form-item>
        <el-input v-model="prizeName" placeholder="奖品名称" clearable></el-input>
      </el-form-item> -->
      <!-- <el-form-item> -->
        <!-- <el-select clearable class="filter-item" style="width: 100%" v-model="dataForm.activityId" placeholder="请选择活动"> -->
          <!-- <el-option v-for="room in activityList" :key="room.id" :label="room.roomName" :value="room.id">
          </el-option> -->
          <!-- <el-option label="活动一" :key="1" value="活动一"></el-option> -->
          <!-- <el-option label="活动二" :key="2" value="活动二"></!-->
        <!-- </el-select> -->
      <!-- </el-form-item> -->
      <el-form-item>
        <!-- <el-select clearable class="filter-item" style="width: 100%" v-model="dataForm.prizeId" placeholder="请选择奖品"> -->
          <!-- <el-option v-for="room in prizeList" :key="room.id" :label="room.roomName" :value="room.id">
          </el-option> -->
          <!-- <el-option label="奖品一" :key="1" :value="1"></el-option> -->
          <!-- <el-option label="奖品二" :key="2" :value="2"></el-option> -->
        <!-- </el-select> -->
      </el-form-item>
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
        width="100"
        label="活动奖品id">
      </el-table-column>
      <el-table-column
        prop="prizeName"
        header-align="center"
        align="center"
        label="奖品名称">
      </el-table-column>
      <el-table-column
        prop="prizeCode"
        header-align="center"
        align="center"
        label="奖品code">
      </el-table-column>
      <el-table-column
        prop="prizeNumber"
        header-align="center"
        align="center"
        width="180"
        label="奖品库存">
      </el-table-column>
      <el-table-column
        prop="generateCodeCount"
        header-align="center"
        align="center"
        label="生成码数量">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="160"
        label="操作">
        <template slot-scope="scope">
          <el-button type="primary" size="small" @click="getPrizeList(scope.row)">详情</el-button>
          <el-button type="success" size="small" @click="generateCode(scope.row)">生成码</el-button>
          <!-- <el-button type="danger" size="small" @click="deleteHandle(scope.row.roleId)">删除</el-button> -->
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
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getActivityPrizeList(activityId2,activityName2)"></add-or-update>
    <generateCode v-if="generateCodeVisible" ref="generateCode" @refreshDataList="getActivityPrizeList(activityId2,activityName2)"></generateCode>
  </div>
  <div v-else>
    <codeBatchJob v-if="prizeListVisible" ref="prizeList" @refreshDataList="getActivityPrizeList(activityId2,activityName2)"></codeBatchJob>
  </div>
</template>

<script>
  import AddOrUpdate from './activity-prize-add-or-update'
  import codeBatchJob from './codeBatchJob'
  import generateCode from './generateCode'
  export default {
     name: 'page3',
     props: {
      activityId: "",
      activityName:""
    },
    data () {
      return {
        dataForm: {
          activityId:1,
          prizeId:1,
        },
        prizeName:'',
        dataList: [{roleId:1,roleName:"1111",remark:"121212"}],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        prizeListVisible:false,
        generateCodeVisible:false,
        pageStatus: 'list',
        activityId2:null,
        activityName2:''
      }
    },
    components: {
      AddOrUpdate,
      codeBatchJob,
      generateCode
    },
    activated () {
      // this.getDataList()
    },
    methods: {
      // 获取数据列表
      getActivityPrizeList (activityId,activityName) {
        this.$loading.open()
        this.activityId2 = activityId;
        this.activityName2 = activityName;
        this.pageStatus = "list";
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/exchange-code/activityPrize/getPageList'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'pageSize': this.pageSize,
            'activityId': activityId
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.data.list
            this.totalPage = data.data.total;
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
      generateCode(row){
        this.generateCodeVisible = true
         this.$nextTick(() => {
          this.$refs.generateCode.init(this.activityId2,this.activityName2,row.prizeId,row.prizeName)
        })
      },
       getPrizeList(row){
         this.pageStatus = "batch";
         this.prizeListVisible = true;
        this.$nextTick(() => {
          this.$refs.prizeList.getBatchList(this.activityId2,row.prizeId)
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
