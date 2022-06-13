<template>
<div class="app-container home">
    <el-row :gutter="20">
      <el-col :sm="24" :lg="24">
        <el-card class="update-log" style="width: 80%">
        <blockquote class="text-warning" style="font-size: 14px">

        </blockquote>
        <h1>任务通知</h1>
        <hr />
          <el-table v-loading="runtimeLoading" :data="runtimeList">
            <el-table-column label="序号" align="center" prop="id" width="100"/>
            <el-table-column label="流程名称" align="center" prop="processName"/>
            <el-table-column label="发起人" align="center" prop="createBy" />
            <el-table-column label="发起时间" align="center" prop="createTime" >
              <template slot-scope="scope">
                <span>{{ parseTime(scope.row.createTime, '{y}-{m}-{d} {h}:{i}:{s}') }}</span>
              </template>
            </el-table-column>
            <el-table-column label="操作" align="center">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="text"
                  icon="el-icon-edit"
                  @click="handleApprove(scope.row)"
                  v-hasPermi="['system:runtime:edit']"
                >审批</el-button>
              </template>
            </el-table-column>
          </el-table>

          <pagination
            v-show="totalR>0"
            :total="totalR"
            :page.sync="queryParamsR.pageNum"
            :limit.sync="queryParamsR.pageSize"
            @pagination="getList"
          />
        </el-card>
      </el-col>
    </el-row>
    <el-divider />
   <el-row :gutter="20">
      <el-col :xs="24" :sm="24">
        <el-card  class="update-log" style="width: 80%">
          <h1 >公告</h1>
          <hr />
          <el-table v-loading="loading" :data="noticeList">
            <el-table-column label="序号" align="center" prop="noticeId" width="100" />
            <el-table-column
              label="公告标题"
              align="center"
              prop="noticeTitle"
              :show-overflow-tooltip="true"
            />
            <el-table-column label="公告类型" align="center" prop="noticeType" width="100">
              <template slot-scope="scope">
                <dict-tag :options="dict.type.sys_notice_type" :value="scope.row.noticeType"/>
              </template>
            </el-table-column>
            <el-table-column label="状态" align="center" prop="status" width="100">
              <template slot-scope="scope">
                <dict-tag :options="dict.type.sys_notice_status" :value="scope.row.status"/>
              </template>
            </el-table-column>
            <el-table-column label="创建者" align="center" prop="createBy" width="100" />
            <el-table-column label="创建时间" align="center" prop="createTime" width="200">
              <template slot-scope="scope">
                <span>{{ parseTime(scope.row.createTime, '{y}-{m}-{d} {h}:{i}:{s}') }}</span>
              </template>
            </el-table-column>
            <el-table-column label="操作" align="center"  width="200">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="text"
                  icon="el-icon-edit"
                  @click="handleUpdate(scope.row)"
                  v-hasPermi="['system:notice:edit']"
                >查 看</el-button>
              </template>
            </el-table-column>
          </el-table>
          <pagination
            v-show="total>0"
            :total="total"
            :page.sync="queryParams.pageNum"
            :limit.sync="queryParams.pageSize"
            @pagination="getList"
          />
        </el-card>
      </el-col>
    </el-row>

  <el-dialog :title="title" :visible.sync="open" width="780px" append-to-body>
    <el-form ref="form" label-width="80px">
      <el-row>
        <el-col :span="24">
          <el-form-item label="内容">
            <editor v-model="noticeContent"  :min-height="192"/>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="cancel">返 回</el-button>
    </div>
  </el-dialog>


  <el-dialog :title="titleR" :visible.sync="openR" width="780px" append-to-body>
    <div ref="FROM_VIEW">
      {{formDetail}}
    </div>
    <div slot="footer" class="dialog-footer">
      <el-button @click="cancel">返 回</el-button>
    </div>
  </el-dialog>
  </div>
</template>

<script>
import {getNotice, listNotice} from "@/api/system/notice";
import {listRuntime, listRuntimeByApprover,getRuntime} from "@/api/system/runtime";
let Base64 = require("js-base64").Base64;
export default {
  name: "Index",
  dicts: ['sys_notice_status', 'sys_notice_type'],
  created() {
    this.getList();
    this.getRunTimeList();
  },
  data() {
    return {
      formDetail:"",
      // 流程实例对象
      processRunTime: {
        id:""
      },
      // 流程实例表格数据
      runtimeList: [],
      runtimeLoading:true,
      // 遮罩层
      loading: true,
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 显示搜索条件
      showSearch: true,
      //runtime总条数
      totalR: 0,
      // 总条数
      total: 0,
      // 公告表格数据
      noticeList: [],
      // 弹出层标题
      title: "",
      titleR: "",
      // 是否显示弹出层
      open: false,
      openR: false,
      //流程查询参数 queryParamsR
      queryParamsR: {
        pageNum: 1,
        pageSize: 10,
        noticeTitle: undefined,
        createBy: undefined,
        status: undefined
      },
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        noticeTitle: undefined,
        createBy: undefined,
        status: undefined
      },
      // 版本号
      version: "3.8.2",
      noticeContent:""
    };
  },
  // 表单参数
  form: {},
  methods: {
    goTarget(href) {
      window.open(href, "_blank");
    },
    /** 查询流程实例列表 */
    getRunTimeList() {
      this.loading = true;
      listRuntimeByApprover(this.queryParamsR).then(response => {
        this.runtimeList = response.rows;
        this.totalR = response.total;
        this.runtimeLoading = false;
      });
    },
    /** 查询公告列表 */
    getList() {
      this.loading = true;
      listNotice(this.queryParams).then(response => {
        this.noticeList = response.rows;
        this.total = response.total;
        this.loading = false;
      });
    },
    // 取消按钮
    cancel() {
      this.open = false;
      this.reset();
    },
    // 表单重置
    reset() {
      this.form = {
        noticeId: undefined,
        noticeTitle: undefined,
        noticeType: undefined,
        noticeContent: undefined,
        status: "0"
      };
      this.resetForm("form");
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      const noticeId = row.noticeId || this.ids
      getNotice(noticeId).then(response => {
        /*this.form = response.data;*/
        this.noticeContent=response.data.noticeContent;
        this.open = true;
        this.title = "公告内容";
      });
    },
    handleApprove(row) {
      this.reset();
      const id = row.id || this.ids
      this.processRunTime.id=id;
      getRuntime(id).then(response => {
        this.formDetail = Base64.fromBase64(response.data.formDetail);
        console.log(response.data);
        console.log("-------------------formDetail-------------------");
        console.log(formDetail);
        this.openR = true;
        this.titleR = "审核流程";
      });
    },
  },
};
</script>

<style scoped lang="scss">
H1,h2,h3,h4{font-size:20px; font-weight:bold}
.home {
  background-image: url("../assets/images/background.jpg");
  blockquote {
    padding: 10px 20px;
    margin: 0 0 20px;
    font-size: 17.5px;
    border-left: 5px solid #eee;
  }
  hr {
    margin-top: 20px;
    margin-bottom: 20px;
    border: 0;
    border-top: 1px solid #eee;
  }
  .col-item {
    margin-bottom: 20px;
  }

  ul {
    padding: 0;
    margin: 0;
  }

  font-family: "open sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
  font-size: 13px;
  color: #676a6c;
  overflow-x: hidden;

  ul {
    list-style-type: none;
  }

  h4 {
    margin-top: 0px;
  }

  h2 {
    margin-top: 10px;
    font-size: 26px;
    font-weight: 100;
  }

  p {
    margin-top: 10px;

    b {
      font-weight: 700;
    }
  }

  .update-log {
    ol {
      display: block;
      list-style-type: decimal;
      margin-block-start: 1em;
      margin-block-end: 1em;
      margin-inline-start: 0;
      margin-inline-end: 0;
      padding-inline-start: 40px;
    }
  }
}
</style>

