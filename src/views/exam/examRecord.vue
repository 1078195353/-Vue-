<template>
  <div class="app-container">
    <el-form :inline="true" :model="filters" class="demo-form-inline">
          <el-form-item label="考试名称">
            <el-input v-model="filters.keyword" placeholder="输入考试名称查询"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" icon="el-icon-search" @click="queryData">查询</el-button>
          </el-form-item>
    </el-form>
    
    <el-table
      v-loading="listLoading"
      :data="list"
      element-loading-text="Loading"
      border
      fit
      highlight-current-row
    >
      <el-table-column
              label="序号"
              type="index"
              width="50"
              align="center">
          <template slot-scope="scope">
              <span>{{(startPage - 1) * pageSize + scope.$index + 1}}</span>
          </template>
      </el-table-column>
      <el-table-column label="姓名"   align="center">
        <template slot-scope="scope">
          {{ scope.row.username }}
        </template>
      </el-table-column>
      <el-table-column label="考试名称"  align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.examName }}</span>
        </template>
      </el-table-column>
      <el-table-column label="得分"  align="center">
        <template slot-scope="scope">
          {{ scope.row.finalScore }}
        </template>
      </el-table-column>
      <el-table-column label="分数等级" align="center">
        <template slot-scope="scope">
          {{ scope.row.resultLevel }}
        </template>
      </el-table-column>
      <el-table-column align="center" label="交卷时间">
        <template slot-scope="scope">
          <i class="el-icon-time" />
          <span>{{ scope.row.finishTime }}</span>
        </template>
      </el-table-column>
      <el-table-column label="考试实际时间(分钟)" align="center">
        <template slot-scope="scope">
          {{ scope.row.costTime }}
        </template>
      </el-table-column>
      <el-table-column
            align="center"
            fixed="right"
            label="操作">
            <template slot-scope="scope">
              <el-button
                size="mini"
                type="primary" icon="el-icon-view"
                @click="handleView(scope.row.id)">查看试卷详情</el-button>
            </template>
      </el-table-column>
    </el-table>

    <el-col>
    		<div class="block" style="float: right;margin-right: 10px;margin-top: 10px;">
    			<el-pagination
    				@size-change="handleSizeChange"
    				@current-change="handleCurrentChange"
    				:current-page="startPage"
    				:page-sizes="pageSizes"
    				:page-size="pageSize"
    				layout="total, sizes, prev, pager, next, jumper"
    				:total="total">
    			</el-pagination>
    	 </div>
    </el-col>

  </div>
</template>

<script>
import { getExamRecordList,getAllExamRecord} from '@/api/exam'
import { getUserId ,getUsername} from '@/utils/auth'
import { getUserList } from '@/api/user'

export default {
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'gray',
        deleted: 'danger'
      }
      return statusMap[status]
    }
  },
  data() {
    return {
      filters: {
      	keyword: ''
      },
      listLoading: false, // 加载等待
      pageSizes: [6, 10, 20, 40],
      startPage: 1,
      pageSize: 6,
      total: 0,
      list: [],
      userList:null,
      examrecodAll:null,
      showAll:false,
    }
  },
  created() {
    this.init()
    setTimeout(()=>{
      this.fetchData();
    },1000)
  },
  computed:{
    userName(){
      return getUsername()
    }
  },
  methods: {
    init(){
      let params = {
        startPage: this.startPage,
        pageSize: this.pageSize,
        username: ''
      }
      getUserList(params).then(res => {
        this.userList = res.data.data
        this.userList.forEach(item => {
          if(item.username === this.userName && (item.roleName === '老师' || item.roleName === '管理员')){
            this.showAll = true;
          }
        })
      })
    },

  fetchData() {
    if(this.showAll){
      this.startPage = 1
      this.pageSize = 100
      this.userList.forEach(item => {
        this.listLoading = true
        let params = {
          startPage: this.startPage,
          pageSize: this.pageSize,
          userId: item.id,
          name: this.filters.keyword
        }
        getExamRecordList(params).then(response => {
          response.data.data.forEach(item => {
            item.finishTime = this.setTime(item.finishTime)
          })
          this.list = this.list.concat(response.data.data)
          this.total += response.data.total
          this.listLoading = false
        }).catch(err =>{
          this.listLoading = false
        })
      })
    } else {
      this.listLoading = true
      let params = {
        startPage: this.startPage,
        pageSize: this.pageSize,
        userId: getUserId(),
        name: this.filters.keyword
      }
      getExamRecordList(params).then(response => {
        response.data.data.forEach(item => {
          item.finishTime = this.setTime(item.finishTime)
        })
        this.list = response.data.data
        this.total = response.data.total
        this.listLoading = false
      }).catch(err =>{
        this.listLoading = false
      })
    }
    
  },
    queryData(){
      this.fetchData()
    },
    // 处理时间
    setTime(date) {
      var dates = new Date(date).toJSON();
      return new Date(+new Date(dates) + 8 * 3600 * 1000).toISOString().replace(/T/g, ' ').replace(/\.[\d]{3}Z/, '') 
    },
    //查看试卷详情
    handleView(recordId){
      this.$router.push({path: '/exam/record/viewAnswer', query: {recordId: recordId,isPractice: true}})
    },

    // 每页大小改变时触发
    handleSizeChange (val) {
    	this.pageSize = val
    	this.fetchData()
    },
    // 当前页码改变时触发
    handleCurrentChange (val) {
    	this.startPage = val
    	this.fetchData()
    }


  }
}
</script>
