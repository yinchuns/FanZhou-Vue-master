<template>
  <div>
  <el-form ref="submitForm" size="small" :inline="true" v-show="showSumb"  label-width="68px">
    <el-button type="primary"    plain
               icon="el-icon-plus"  @click="submitForm()">提交</el-button>
  </el-form>

    <el-form ref="submitFormA" size="small" :inline="true" v-show="showApprove" label-width="68px">
      <el-form-item label="审批意见" prop="approveMsg">
        <el-input   v-model="approveMsg" type="textarea"  :autosize="{minRows: 4, maxRows: 4}" :style="{width: '200%'}" placeholder="请输入审批意见"/>
      </el-form-item><br>
      <el-button type="primary"    plain
                 icon="el-icon-plus"  @click="submitForm(1)">通过</el-button>
      <el-button type="primary"    plain
                 icon="el-icon-plus"  @click="submitForm(2)">退回</el-button>
    </el-form><br>

    <el-row :gutter="10" >
      <el-col :span="1.5" v-for="(item,index) of nodeList" >
        <div style="display: flex;align-items: center;flex-wrap: wrap;">
          <div :class="item.checkStatus==1?'flowCheckBox':'flowBox'">{{item.nodeName}}</div>
          <div v-if="index < (nodeList.length-1)" >{{"——>"}}</div>
        </div>
      </el-col>
    </el-row>

    <el-table v-loading="loadingNodeS" :data="noticeList">
      <el-table-column label="节点名称" align="center" prop="nodeName" />
      <el-table-column label="审批时间 " align="center" prop="approveTime" width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.approveTime, '{y}-{m}-{d} {h}:{i}:{s}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="审批人" align="center" prop="approver" />
      <el-table-column label="审批意见  " align="center" prop="approveMsg" />
      <el-table-column label="审批状态 " align="center" >
        <template slot-scope="scope">
          <span v-if="scope.row.approveStatus===1">通过</span>
          <span v-else-if="scope.row.approveStatus===2">驳回</span>
          <span v-else-if="scope.row.approveStatus===3">提交</span>
        </template>
      </el-table-column>
    </el-table>

  <!-- 指派节点审核人 -->
  <el-dialog :title="title" :visible.sync="openApprover" width="600px" append-to-body>
    <el-table v-loading="approverLoading" :data="approverList" @current-change="clickChange">
      <el-table-column label="选择" width="55">
        　　　　<template slot-scope="scope">
        　　　　　　<el-radio v-model="tableRadio" :label="scope.row"><i></i></el-radio>
        　　　　</template>
      </el-table-column>
      <el-table-column label="节点审核人" align="center" prop="approverName" />
      <el-table-column label="备注 " align="center" prop="remark" />
    </el-table>
    <el-button type="primary"   plain
               icon="el-icon-plus"
               @click="confirmApprover()">选择</el-button>
  </el-dialog>
  </div>
</template>
<style>
.flowCheckBox{
  border-width: 1px;
  padding: 20px;
  border-style: solid;
  border-color: #ECECEC;
  background-color: #006241;
  color:white;
}

.flowBox{
  border-width: 1px;
  padding: 20px;
  border-style: solid;
  border-color: #ECECEC;
  background-color: #00AE72;
  color:white;
}

</style>


<script>

import { listRuntime, getRuntime, delRuntime, addRuntime, updateRuntime } from "@/api/system/runtime";
import {getRuntimeByFormId} from "../../../api/system/runtime";
import {getAproverlist} from "../../../api/system/approver";
import html2canvas from 'html2canvas';
let Base64 = require("js-base64").Base64;
export default {
  name: "Runtime",
  props:{
    prunTime:{
      type:Object,
      default:''
    }
  },
  data() {
    return {
      title:'',
      approverLoading: false,
      openApprover:false,
      loadingNodeS:false,
      //显示提交按钮
      showSumb: false,
      //显示审核页面
      showApprove: false,
      //节点数据树
      nodeList:[],
      //流程日志列表
      noticeList: [],
      // 流程实例对象
      processRunTime: {
        id:"",
        formId:"",
        processMark:""
      },
      //审核意见
      approveMsg:"",
      //审核人列表
      approverList: [],
      //选中id
      tableRadio:"",
      APPROVER_ID:"",
      rules: {
      }
    };
  },
  async created() {
    await this.reset();
    await this.getProcessRunTime();
  },
  mounted() {
    /*this.reset();
    this.getProcessRunTime();*/
  },
  watch:{
    prunTime:function (){
      this.getProcessRunTime();
    }
  },
  methods: {
    // 表单重置
    reset() {
        //节点数据树
        this.nodeList=[];
        //流程日志列表
        this.noticeList= [];
        //审核意见
        this.approveMsg="";
        //审核人列表
        this.approverList= [];
        //选中id
        this.tableRadio="";
        this.APPROVER_ID="";
        this.processRunTime= {
          id:"",
            formId:"",
            processMark:""
        };
      //显示提交按钮
      this.showSumb=false;
      //显示审核页面
      this.showApprove=false;
    },
    clickChange (item) {
      this.APPROVER_ID = item.approverId;
    },
    /** 选中审核人*/
    async  confirmApprover(){
     // var  item=document.getElementById("FORM_DITAIl");//获取div标签
     /* let dataURL ="";
      // 传入你要生成图片的容器 dom

      await    html2canvas(item).then(canvas => {
        console.log(item);
        // dataURL  生成的图片链接 这里是base64格式的
        dataURL = canvas.toDataURL("image/png");
      });
      console.log("---------------dataURL--------------------");
      await  console.log("aaa:"+dataURL);
      return;*/
      this.processRunTime.approverId=this.APPROVER_ID;
          if(this.prunTime.formId){
           //执行流程提交创建动作
            let params= {
              processMark: this.processRunTime.processMark,
              formId:this.processRunTime.formId,
              approverId:this.APPROVER_ID,
              approveMsg:this.approveMsg
            }
            addRuntime(params).then(response => {
              this.$modal.msgSuccess("成功");
              this.open = false;
              this.getProcessRunTime();
            });
          }else if(this.prunTime.id){
            //执行流程通过动作
            let params= {
              processMark: this.processRunTime.processMark,
              id:this.processRunTime.id,
              approverId:this.APPROVER_ID,
              approveMsg:this.approveMsg
            }
            updateRuntime(params).then(response => {
              this.$modal.msgSuccess("成功");
              this.open = false;
              this.getProcessRunTime();
            });
          }
          this.openApprover = false;

    },
    /** 获取审核人列表 */
    getNodeApproverList() {
      let queryParams= {
        processMark: this.processRunTime.processMark,
        currentNode:this.processRunTime.currentNode
      }
        getAproverlist(queryParams).then(response => {
          if(response.rows){
            this.approverList = response.rows;
            this.approverLoading=false;
          }
        });
    },
    /** 查询流程实例列表 */
    getProcessRunTime() {
      if(this.prunTime.id){
        getRuntime(this.prunTime.id).then(response => {
          if(response.data){
            this.processRunTime = response.data;
            this.nodeList=this.processRunTime.sysProcessNodeList;
            this.noticeList=this.processRunTime.sysProcessNoticeList;
            if(!this.processRunTime.currentNode || this.processRunTime.currentNode==0){
              this.showSumb=true;
              this.showApprove=false;
            }else{
              this.showApprove=true;
              this.showSumb=false;
            }
            //流程已结束
            if(this.processRunTime.status==3){
              this.showSumb=false;
              this.showApprove=false;
            }
          }else{
            this.processRunTime =this.prunTime;
          }
        });

      }else if(this.prunTime.formId){
        this.showSumb=true;
        this.showApprove=false;
        this.processRunTime =this.prunTime;
      }
    },
    /** 提交按钮 */
    submitForm(status) {
          if (this.prunTime.id) {
            this.processRunTime.approveMsg=this.approveMsg;
            if(status==2){//退回
              this.processRunTime.status=status;
              //执行退回动作
              let params= {
                processMark: this.processRunTime.processMark,
                id:this.processRunTime.id,
                approverId:this.APPROVER_ID,
                status:status,
                approveMsg:this.approveMsg
              }
              updateRuntime(params).then(response => {
                this.$modal.msgSuccess("修改成功");
                this.open = false;
                this.getProcessRunTime();
              });
            }else{
              //判断是否是最后一个节点
              if(this.processRunTime.currentNode==this.processRunTime.maxNode || this.processRunTime.currentNode==0){
                //执行流程通过动作
                let params= {
                  processMark: this.processRunTime.processMark,
                  id:this.processRunTime.id,
                  approverId:this.APPROVER_ID,
                  approveMsg:this.approveMsg
                }
                updateRuntime(params).then(response => {
                  this.$modal.msgSuccess("修改成功");
                  this.open = false;
                  this.getProcessRunTime();
                });
              }else{
                //获取审核人弹框，选择审核人
                this.openApprover = true;
                this.approverLoading=true;
                this.title = "选择审核人";
                this.getNodeApproverList();
              }
            }
          } else {
            //选择审核人弹框，选择审核人后进行下一步操作
            this.openApprover = true;
            this.approverLoading=true;
            this.title = "选择审核人";
            this.getNodeApproverList();
          }
    }
  }
};
</script>
