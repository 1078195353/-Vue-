<template>
  <div class="app-container">

      <el-tabs v-model="activeName" @tab-click="handleClick">
        <el-tab-pane label="填空题" name="first"></el-tab-pane>
        <el-tab-pane label="判断题" name="second"></el-tab-pane>
        <el-tab-pane label="单选题" name="third"></el-tab-pane>
        <el-tab-pane label="多选题" name="fourth"></el-tab-pane>
      </el-tabs>

    <el-form :inline="true" :model="filters" class="demo-form-inline">
          <el-form-item label="题目内容">
            <el-input v-model="filters.keyword" placeholder="输入题目内容查询"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" icon="el-icon-search" @click="queryData">查询</el-button>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" icon="el-icon-plus" @click="handelAdd">新增题目</el-button>
          </el-form-item>
          <el-form-item>
          	<el-button type="primary" round @click="exportQuestionTemplate">下载导入模板</el-button>
          </el-form-item>
          <el-form-item>
				   <el-upload class="upload-demo"
                 :action="uploadUrl"
								 :before-upload="handleBeforeUpload"
								 :on-error="handleUploadError"
                 :show-file-list="showFile"
								 accept=".xls,.xlsx"
                 :http-request="uploadFile"
								 :file-list="fileList">
				      <el-button type="infor" round>导入题库</el-button>
				   </el-upload>
          </el-form-item>
          <el-form-item>
          	<el-button type="infor" round @click="exportQuestions">导出题库</el-button>
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
      <el-table-column label="题目内容"  align="center" :show-overflow-tooltip="true">
        <template slot-scope="scope">
          <span v-html="scope.row.questionContent"></span>
        </template>
      </el-table-column>
      <el-table-column label="创建时间"  align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.createTime }}</span>
        </template>
      </el-table-column>
<!--      <el-table-column label="更新时间"  align="center">
        <template slot-scope="scope">
          {{ scope.row.updateTime }}
        </template>
      </el-table-column> -->
      <el-table-column label="难度等级" align="center">
        <template slot-scope="scope">
          {{ scope.row.questionLevelStr }}
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
                @click="handleView(scope.row)">查看题目
              </el-button>
              <el-button
                size="mini"
                type="danger" icon="el-icon-delete"
                @click="deleteQuestion(scope.row)">删除题目
              </el-button>
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

   <!-- 　　新增和编辑 弹框 -->
    <el-dialog
        :title="titleMap[currentType]"
        :visible.sync="dialogFormVisible" >
          <div style="width:80%;margin: 0 auto">
            <el-form :model="ruleForm" status-icon ref="ruleForm" :inline="false" label-width="90px" class="demo-ruleForm">
                <el-form-item label="题目类型">
                  <el-select v-model="activeName" placeholder="请选择题型" @blur="changeType" :disabled="currentType=='view'">
                    <el-option
                      v-for="item in options"
                      :key="item.value"
                      :label="item.label"
                      :value="item.value">
                    </el-option>
                  </el-select>
                </el-form-item>
                <el-form-item label="题目" prop="questionContent" :rules="[{ required: true, message: '请输入题目'}]">
                  <my-editor
                     v-model="ruleForm.questionContent"
                     :isClear="isClear"
                     @change="change"
                  ></my-editor>
<!--                  <el-input type="textarea" placeholder="输入题目,如:从计算机网络系统组成的角度看，计算机网络可以分为()和()。注意需要考生答题部分一定要用括号（英文半角）括起来。"
                      auto-complete="off" v-model="ruleForm.questionContent" :disabled="currentType=='view'"></el-input> -->
                </el-form-item>
                <!-- 填空题 -->
                <template v-if="activeName=='first'">
                  <el-form-item label="正确答案" prop="questionAnswer" :rules="[{ required: true, message: '请输入题目'},{ min: 1, max: 100, message: '长度在 1 到 100 个字符'}]">
                    <el-input  type="text" placeholder="每两个个空之间的答案用英文逗号分隔" auto-complete="off" v-model="ruleForm.questionAnswer" :disabled="currentType=='view'"></el-input>
                  </el-form-item>
                </template>
                <!-- 判断题 -->
                <template v-if="activeName == 'second'" >
                  <el-form-item label="正确答案" prop="questionAnswer" :rules="{ required: true, message: '请选择答案', trigger: 'change' }">
                    <el-select v-model="ruleForm.questionAnswer" filterable placeholder="请选择答案" :disabled="currentType=='view'">
                      <el-option
                        v-for="item in judgeOptions"
                        :key="item.id"
                        :label="item.name"
                        :value="item.id">
                      </el-option>
                   </el-select>
                  </el-form-item>
                </template>
                <!-- 单选、多选公共部分 -->
               <template v-if="activeName == 'third' || activeName == 'fourth'" >
                 <el-form-item label="选项A" prop="choiceA" :rules="[{ required: true, message: '输入选项内容'},{ min: 1, max: 100, message: '长度在 1 到 100 个字符'}]">
                   <el-input  type="text" placeholder="输入选项内容" auto-complete="off" v-model="ruleForm.choiceA" :disabled="currentType=='view'"></el-input>
                 </el-form-item>
                 <el-form-item label="选项B" prop="choiceB" :rules="[{ required: true, message: '输入选项内容'},{ min: 1, max: 100, message: '长度在 1 到 100 个字符'}]">
                   <el-input  type="text" placeholder="输入选项内容" auto-complete="off" v-model="ruleForm.choiceB" :disabled="currentType=='view'"></el-input>
                 </el-form-item>
                 <el-form-item label="选项C" prop="choiceC" :rules="[{ required: true, message: '输入选项内容'},{ min: 1, max: 100, message: '长度在 1 到 100 个字符'}]">
                   <el-input  type="text" placeholder="输入选项内容" auto-complete="off" v-model="ruleForm.choiceC" :disabled="currentType=='view'"></el-input>
                 </el-form-item>
                 <el-form-item label="选项D" prop="choiceD" :rules="[{ required: true, message: '输入选项内容'},{ min: 1, max: 100, message: '长度在 1 到 100 个字符'}]">
                   <el-input  type="text" placeholder="输入选项内容" auto-complete="off" v-model="ruleForm.choiceD" :disabled="currentType=='view'"></el-input>
                 </el-form-item>
               </template>
               <!-- 单选 -->
               <template v-if="activeName == 'third'">
                 <el-form-item  label="正确答案" prop="questionAnswer" :rules="{ required: true, message: '请选择答案', trigger: 'change' }">
                   <el-select v-model="ruleForm.questionAnswer" filterable placeholder="请选择答案" :disabled="currentType=='view'">
                     <el-option
                       v-for="item in singleOptions"
                       :key="item.id"
                       :label="item.name"
                       :value="item.id">
                     </el-option>
                  </el-select>
                 </el-form-item>
               </template>
               <!-- 多选 -->
               <template v-if="activeName == 'fourth'">
                 <el-form-item  label="正确答案" prop="multi" :rules="{ required: true, message: '请选择答案', trigger: 'change' }">
                   <el-select multiple v-model="ruleForm.multi" filterable placeholder="请选择答案" :disabled="currentType=='view'">
                     <el-option
                       v-for="item in singleOptions"
                       :key="item.name"
                       :label="item.name"
                       :value="item.id">
                     </el-option>
                  </el-select>
                 </el-form-item>
               </template>
                <el-form-item label="题目难度" prop="questionLevel" :rules="[{ required: true, message: '请选择题目难度', trigger: 'change' }]">
                  <el-select v-model="ruleForm.questionLevel" filterable placeholder="请选择" :disabled="currentType=='view'">
                    <el-option
                      v-for="item in levels"
                      :key="item.id"
                      :label="item.name"
                      :value="item.id">
                    </el-option>
                 </el-select>
                </el-form-item>
                <el-form-item label="答案解析" prop="questionExplain" :rules="[{max: 200, message: '最多输入 200 个字符',trigger: 'blur' }]">
                  <el-input type="textarea" placeholder="解析答案,方便理解" auto-complete="off" v-model="ruleForm.questionExplain" :disabled="currentType=='view'"></el-input>
                </el-form-item>
              </el-form>
           </div>
           <div slot="footer" class="dialog-footer">
             <template v-if="currentType=='view'">
               <el-button @click="dialogFormVisible = false">关 闭</el-button>
             </template>
             <template v-else>
               <el-button @click="dialogFormVisible = false">取 消</el-button>
               <el-button type="primary" @click="submitForm('ruleForm')">确 定</el-button>
             </template>
           </div>
   </el-dialog>


  </div>
</template>

<script>
import { getQuestionList,addFill,delFill,addJudge,delJudge,addSingle,delSingle,addMulti,delMulti,exportQuestionList,exportTemplate,importQuestions } from '@/api/question'
import { getUserId } from '@/utils/auth'
import MyEditor from '@/components/editor/MyEditor.vue'

export default {
  components:{
    MyEditor
  },
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
      activeName: 'first',
      options: [
        {value:'first',label:'填空题'},
        {value:'second',label:'判断题'},
        {value:'third',label:'单选题'},
        {value:'fourth',label:'多选题'}
      ],
      judgeOptions: [
        {id: 'T',name: '正确'},
        {id: 'F',name: '错误'}
      ],
      singleOptions: [
       {id: 'A',name: 'A'},
       {id: 'B',name: 'B'},
       {id: 'C',name: 'C'},
       {id: 'D',name: 'D'}
      ],
      //题目难度等级，最好是到后台查数据字典，这里简单起见，没有这么做了
      levels: [
        {id: 1,name: '简单'},
        {id: 2,name: '中等'},
        {id: 3,name: '困难'}
      ],
      listLoading: false, // 加载等待
      pageSizes: [6, 10, 20, 40],
      startPage: 1,
      pageSize: 6,
      total: 0,
      list: null,
      listLoading: true,
      currentType: 'add',  //新增、查看、编辑公用一个弹窗，当前弹框类型，'add','view','edit'，默认为新增
      dialogFormVisible:false,
      ruleForm: {
        id: '',
      	questionContent: '',
      	questionAnswer: '',
      	questionLevel: '',
        questionExplain: '',
        multi: [],  //多选题选项答案，用数组临时存储
        choiceA: '',
        choiceB: '',
        choiceC: '',
        choiceD: ''
      },
      //新增or编辑弹框标题(根据点击的新增or编辑按钮显示不同的标题)
      titleMap: {
          add:'新增',
          view: "查看",
          edit: "修改"
      },
      // Excel导入相关属性
      uploadUrl: '/question/import',  // 随便写一个
      fileList: [],
      showFile: false,
      isClear: false,

    }
  },
  created() {
    this.fetchData()
  },
  methods: {
    fetchData() {
      this.listLoading = true
      let params = {
        startPage: this.startPage,
        pageSize: this.pageSize,
        content: this.filters.keyword,
        questionType: this.activeName
      }
      getQuestionList(params).then(response => {
        this.list = response.data.data
        this.list.forEach(item=> {
          item.createTime = this.setTime(item.createTime)
        })
        this.total = response.data.total
        this.listLoading = false
      }).catch(err =>{
        this.listLoading = false
      })
    },
    // 处理时间
    setTime(date) {
      var dates = new Date(date).toJSON();
      return new Date(+new Date(dates) + 8 * 3600 * 1000).toISOString().replace(/T/g, ' ').replace(/\.[\d]{3}Z/, '') 
    },

    queryData(){
      this.fetchData()
    },

    // 获取文本编辑器的内容
    change(val){
      // console.log("editor中，题目内容： "+val)
    },

    //切换标签页
    handleClick(tab, event) {
      this.fetchData()
      // //debugger
      // console.log("tab =="+tab+ " event =="+event)
    },
    //查看题目详情
    handleView(obj){
      this.currentType = 'view'
      //debugger
      this.ruleForm = obj
      if(!obj.questionExplain){
        this.ruleForm.questionExplain = '暂无解析'
      }
      if(this.activeName == 'fourth'){
        //给多选题答案赋值
        this.ruleForm.multi = obj.questionAnswer.split(",")
      }
      //显示弹框
      this.dialogFormVisible = true;
    },
    // 删除题目
    deleteQuestion(obj){
      if(this.activeName == 'first') {
        delFill(obj.id)
      } else if (this.activeName == 'second'){
        delJudge(obj.id)
      } else if (this.activeName == 'third'){
        delSingle(obj.id)
      } else {
        delMulti(obj.id)
      }
      setTimeout(()=>{
        this.fetchData();
      },500)
    },
    //添加
    handelAdd() {
      this.currentType = 'add'
      //显示弹框
      this.dialogFormVisible = true;
     },
     //编辑
    handleEdit(obj) {
      this.currentType = 'edit'
      this.ruleForm = obj
      //显示弹框
      this.dialogFormVisible = true;
     },
     // 导出题库
    exportQuestions(){
        exportQuestionList().then(res => {
          const link = document.createElement('a')
          let blob = new Blob([res.data],{type: 'application/vnd.ms-excel'});
          link.style.display = 'none'
          link.href = URL.createObjectURL(blob);
          link.setAttribute('download','题库.xlsx')
          document.body.appendChild(link)
          link.click()
          document.body.removeChild(link)
        }).catch(e => {
          console.log("导出题库失败")
        })
    },

    // 题库导入模板
    exportQuestionTemplate(){
        exportTemplate().then(res => {
          const link = document.createElement('a')
          let blob = new Blob([res.data],{type: 'application/vnd.ms-excel'});
          link.style.display = 'none'
          link.href = URL.createObjectURL(blob);
          link.setAttribute('download','题库导入模板.xlsx')
          document.body.appendChild(link)
          link.click()
          document.body.removeChild(link)
        }).catch(e => {
          console.log("导出题库失败")
        })
    },

    // 文件上传之前
    handleBeforeUpload(file){
      if(this.fileList.length > 1){
        this.$message.warning("只能上传一个文件");
        return false;
      }
      let fileType = file.name.substring(file.name.lastIndexOf('.') + 1);
      if(fileType == 'xls' || fileType == 'xlsx'){
        this.fileList.push(file);
      }else{
        this.$message.warning("只能上传.xls和.xlsx格式的文件");
        return false;
      }
    },
    // 执行文件上传
    uploadFile(){
      let formData = new FormData();
      formData.append("file", this.fileList[0]);
      const that = this;
      importQuestions(formData).then(res => {
        this.$message.success('文件上传成功');
        // 刷新列表
        that.fetchData();
      }).catch(e => {
        this.$message.error('文件上传失败');
      });
    },
    // 文件上传出错
    handleUploadError(){
       this.$message.error('文件上传失败');
    },

    changeType(val){
      // let t = this.activeName
      //debugger
      // 要清除单选的值，不然切换到多选时只能 只读，不能修改，
      this.ruleForm.questionAnswer = ''
      // console.log("当前值 "+val)

    },
    // 表单提交
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
      	if (valid) {
          // 新增
          if(this.currentType == 'add'){
              // //debugger
              let doAdd = addFill
              if(this.activeName == 'second'){
                doAdd = addJudge
              }else if(this.activeName == 'third'){
                doAdd = addSingle
              }else if(this.activeName == 'fourth'){
               // console.log("多选题答案  "+this.multi)
               let answer = this.ruleForm.multi
               this.ruleForm.questionAnswer = answer.join(",")
               doAdd = addMulti
              }
              //这里主要是为了复用代码，没有分别写一个函数，通过这种方式，可以动态改变要调用的方法
              doAdd(this.ruleForm).then(response => {
                // 不管成功还是失败，清除答案
               // this.ruleForm.questionAnswer = ''
                let msg = "添加失败"
                let msgType = 'error'
                if(response.data.code == 200){
                  msg = "添加成功"
                  msgType = 'success'
                }
                this.$message({
                          message: msg,
                          type: msgType
                        });
                // 关闭弹窗
                this.dialogFormVisible = false
                //刷新列表
                this.fetchData()
              }).catch(err =>{
                this.$message({
                          message: '添加失败',
                          type: 'error'
                        });
              })

          }else if(this.currentType == 'edit'){
           // this.doUpdate()
          }
      	} else {
      		this.$message({message: '表单有必填项没填',
      		      type: 'error'});
      		return false
      	}
      })

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
