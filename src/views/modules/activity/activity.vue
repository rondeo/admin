<template>
  <div class="mod-role" v-if="pageStatus==='list'">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="temp.activityName" placeholder="活动名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border fit highlight-current-row
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        width="80"
        label="活动ID">
      </el-table-column>
      <el-table-column
        prop="activityCode"
        header-align="center"
        align="center"
        label="活动code">
      </el-table-column>
      <el-table-column
        prop="activityName"
        header-align="center"
        align="center"
        label="活动名称">
      </el-table-column>
      <el-table-column
        header-align="center"
        align="center"
        width="250"
        label="活动时间">
        <template slot-scope="scope">
            <span>{{scope.row.activityTimeStart | formatDate}}~~{{scope.row.activityTimeEnd | formatDate}}</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="remark"
        header-align="center"
        align="center"
        label="活动状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status === 1" size="small" type="danger">未公开</el-tag>
          <el-tag v-else size="small">已公开</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="primary" size="small" @click="activityDetailHandle(scope.row)">详情</el-button>
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
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
  <div v-else>
    <br>
    <!-- <el-card class="box-card"> -->
       <!-- <el-button plain @click="handleCancel">返回</el-button> -->
      <!-- <h2 style="margin-top:0;">活动信息</h2>
      <el-row :gutter="20">
          <el-col :span="5">活动名称: {{temp.mpName}}</el-col>
          <el-col :span="5">活动code: {{temp.mpAlias}}</el-col>
          <el-col :span="5">活动时间: 2018-09-04</el-col>
      </el-row>
      <hr> -->
      <el-row :gutter="20">
        <el-col :span="24">
          <el-tabs @tab-click="routerTo" class="tab">
            <el-tab-pane label="活动详情">
              <page1 v-if="isPage1" ref="page1" v-bind:activityId="temp.activityId" @refreshDataList="getDataList"></page1>
            </el-tab-pane>
            <el-tab-pane label="活动区域">
              <page2 v-if="isPage2" ref="page2" v-bind:activityName="temp.activityName" v-bind:activityId="temp.activityId" @refreshDataList="getDataList"></page2>
            </el-tab-pane>
            <el-tab-pane label="活动奖品">
              <page3 v-if="isPage3" ref="page3" v-bind:activityName="temp.activityName" v-bind:activityId="temp.activityId" @refreshDataList="getDataList"></page3>
            </el-tab-pane>
          </el-tabs>
        </el-col>
      </el-row>
    <!-- </el-card> -->
  </div>
</template>

<script>
  import { formatDate } from '@/utils'
  import AddOrUpdate from './activity-add-or-update'
  import page1 from './page1'
  import page2 from './page2'
  import page3 from './page3'
  export default {
    data () {
      return {
        dataForm: {
          roleName: ''
        },
        activityId:null,
        temp:{
          activityId:null,
          activityName:'',
          activityCode:''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        pageStatus:'list',
        isPage1:false,
        isPage2:false,
        isPage3:false
      }
    },
    components: {
      AddOrUpdate,
      page1,
      page2,
      page3
    },
    filters: {
      formatDate(time) {
        var date = new Date(time)
        return formatDate(date, 'yyyy-MM-dd')
      }
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.$loading.open()
        this.dataListLoading = true
        this.pageStatus = 'list'
        this.$http({
          url: this.$http.adornUrl('/exchange-code/activity/getActivityPageList'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'pageSize': this.pageSize,
            'activityName': this.temp.activityName,
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
          this.$refs.addOrUpdate.init(id)
        })
      },
       // 活动详情
      activityDetailHandle (row) {
        this.pageStatus = 'detail'
        this.temp.activityId = row.id;
        this.temp.activityName = row.activityName;
        this.temp.activityCode = row.activityCode;
        let data = {};
        data.index = 0;
        this.routerTo(data,row.id);
      },
      routerTo(val){
          if(val.index == 0){
            this.isPage1 = true;
            setTimeout(() => {
                  this.$refs.page1.init(this.temp.activityId);
            }, 500);
          }
          if(val.index == 1){
            this.isPage2 = true;
             setTimeout(() => {
                  this.$refs.page2.getAddressList(this.temp.activityId,this.temp.activityName);
            }, 500);
          }
          if(val.index == 2){
            this.isPage3 = true;
             setTimeout(() => {
                  this.$refs.page3.getActivityPrizeList(this.temp.activityId,this.temp.activityName);
            }, 500);
          }
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
          // this.$http({
          //   url: this.$http.adornUrl('/exchange-code/activity/deleteActivity'),
          //   method: 'post',
          //   data: this.$http.adornData(ids, false)
          // }).then(({data}) => {
          //   if (data && data.code === 0) {
          //     this.$message({
          //       message: '操作成功',
          //       type: 'success',
          //       duration: 1500,
          //       onClose: () => {
          //         this.getDataList()
          //       }
          //     })
          //   } else {
          //     this.$message.error(data.msg)
          //   }
          // })
        }).catch(() => {})
      }
    }
  }
</script>
