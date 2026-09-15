<template>
   <div class="app-container">
     <el-form :model="queryParams" ref="queryRef" :inline="true">
       <el-form-item label="执行器" prop="jobGroup">
         <el-select v-model="queryParams.jobGroup" placeholder="执行器" filterable style='width: 200px' @change="handleQuery">
           <el-option v-for="item in appOptions" :key="item.id" :label="item.title" :value="item.id"/>
         </el-select>
       </el-form-item>
       <el-form-item label="状态" prop="triggerStatus">
         <el-select v-model="queryParams.triggerStatus" placeholder="状态" filterable style='width: 100px' @change="handleQuery">
           <el-option v-for="dict in TriggerStatus" :key="dict.value" :label="dict.label" :value="dict.value"/>
         </el-select>
       </el-form-item>
       <el-form-item label="任务描述" prop="jobDesc">
         <el-input v-model="queryParams.jobDesc" placeholder="任务描述" clearable style="width: 160px" @keyup.enter="handleQuery"/>
       </el-form-item>
       <el-form-item label="Hander" prop="executorHandler">
         <el-input v-model="queryParams.executorHandler" placeholder="JobHander" clearable style="width: 160px" @keyup.enter="handleQuery"/>
       </el-form-item>
       <el-form-item label="负责人" prop="author">
         <el-input v-model="queryParams.author" placeholder="负责人" clearable style="width: 120px" @keyup.enter="handleQuery"/>
       </el-form-item>
       <el-form-item>
         <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
         <el-button icon="Refresh" @click="resetQuery">重置</el-button>
       </el-form-item>
       <el-form-item style="float: right;">
         <el-button type="primary" plain icon="Plus" @click="handleAdd">新增</el-button>
       </el-form-item>
     </el-form>
     <el-table v-loading="loading" :data="dataList">
       <el-table-column label="ID" prop="id" width="80"/>
       <el-table-column label="任务描述" prop="jobDesc" min-width="200"/>
       <el-table-column label="调度类型" prop="jobDesc" min-width="200">
         <template #default="{row}">
           <span>{{row.scheduleType}}: {{row.scheduleConf}}</span>
         </template>
       </el-table-column>
       <el-table-column label="运行模式" prop="jobDesc" min-width="200">
         <template #default="{row}">
           <div>
             <span v-if="row.glueType !== 'BEAN'"><dict-tag :options="GlueType" :value="row.glueType"/></span>
             <span v-if="row.glueType === 'BEAN'">{{row.glueType}}: {{row.executorHandler}}</span>
           </div>
         </template>
       </el-table-column>
       <el-table-column label="负责人" prop="author" min-width="120"/>
       <el-table-column label="状态" prop="triggerStatus" width="80" fixed='right'>
         <template #default="{row}">
           <el-switch v-model="row.triggerStatus" :active-value="1" :inactive-value="0" size="small" @click="changeTriggerStatus(row)"/>
         </template>
       </el-table-column>

       <el-table-column label="操作" fixed='right' width="270">
         <template #default="{row}">
           <el-button link type="primary" icon="VideoPlay" @click="handleExec(row)">执行</el-button>
           <el-button link type="primary" icon="Notebook" @click="handleLog(row)">日志</el-button>
           <el-button link type="primary" icon="Edit" @click="handleUpdate(row)">编辑</el-button>
           <el-dropdown @command="(command) => handleCommand(command, row)" trigger="hover">
             <el-button link type="primary" style="padding-left: 12px;">更多<el-icon><DArrowRight /></el-icon></el-button>
             <template #dropdown>
               <el-dropdown-item command="handleCopy" icon="DocumentCopy">复制</el-dropdown-item>
               <el-dropdown-item command="handleReg" icon="Promotion">注册节点</el-dropdown-item>
               <el-dropdown-item command="handleNexttime" icon="Clock">下次执行时间</el-dropdown-item>
               <el-dropdown-item command="handleGlueIde" icon="Edit" v-if="row.glueType !== 'BEAN'">GLUE IDE</el-dropdown-item>
               <el-dropdown-item command="handleDelete" icon="DeleteFilled">删除</el-dropdown-item>
             </template>
           </el-dropdown>
         </template>
       </el-table-column>
     </el-table>

     <pagination
         v-show="total > 0"
         :total="total"
         v-model:page="queryParams.current"
         v-model:limit="queryParams.size"
         @pagination="getList"
     />

     <edit ref="editRef" @change="getList"/>
     <exec ref="execRef"/>
     <reg ref="regRef"/>
     <next-tigger-time ref="nextTiggerTimeRef"/>
     <glue-ide ref="glueIdeRef"/>
   </div>
</template>

<script>
import {jobinfoPage, jobinfoRemove, jobinfoStart, jobinfoStop} from "@/api/jobinfo";
import {jobgroupPage} from "@/api/jobgroup";
import Edit from "./components/edit"
import GlueIde from "./components/glueIde"
import Exec from "./components/exec"
import Reg from "./components/reg"
import NextTiggerTime from "./components/nextTiggerTime"
import GlueType from "@/api/dict/GlueType.json"
import TriggerStatus from "@/api/dict/TriggerStatus.json"

export default {
  name: "Jobinfo",
  components: { Edit, GlueIde, Exec, Reg, NextTiggerTime },
  data() {
    return {
      appOptions: [],
      dataList: [],
      loading: false,
      total: 0,
      queryParams: {
        start: 0,
        length: 10,
        current: 1,
        size: 10,
        jobGroup: 1,
        triggerStatus: -1,
        jobDesc: '',
        executorHandler: '',
        author: '',
      },
      GlueType,
      TriggerStatus,
    };
  },
  created() {
    this.init();
  },
  methods: {
    init() {
      this.getApps();
    },
    getApps() {
      jobgroupPage({
        start: 0,
        length: 10000,
        appname: '',
        title: ''
      }).then(res => {
        this.appOptions = res.data;
        if (this.appOptions && this.appOptions.length > 0) {
          this.queryParams.jobGroup = this.appOptions[0].id;
          this.getList();
        }
      });
    },
    /** 查询参数列表 */
    getList() {
      if (!this.queryParams.jobGroup) {
        return;
      }
      this.loading = true;
      this.queryParams.start = (this.queryParams.current - 1) * this.queryParams.size;
      jobinfoPage(this.queryParams).then(res => {
        this.dataList = res.data;
        // res.recordsFiltered
        this.total = res.recordsTotal;
      }).finally(() => {
        this.loading = false;
      });
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParams.current = 1;
      this.getList();
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.resetForm("queryRef");
      this.handleQuery();
    },
    changeTriggerStatus(row) {
      if (row.triggerStatus === 1) {
        jobinfoStart({id: row.id}).then(res => {
          this.$modal.msgSuccess("开启成功！");
        }).catch(() => {
          this.$modal.msgError("开启失败！");
          row.triggerStatus = 0;
        });
      }
      if (row.triggerStatus === 0) {
        jobinfoStop({id: row.id}).then(res => {
          this.$modal.msgSuccess("关闭成功！");
        }).catch(() => {
          this.$modal.msgError("关闭失败！");
          row.triggerStatus = 1;
        });
      }
    },
    handleCommand(command, row) {
      switch (command) {
        case "handleCopy":
          this.handleCopy(row);
          break;
        case "handleReg":
          this.handleReg(row);
          break;
        case "handleNexttime":
          this.handleNexttime(row);
          break;
        case "handleGlueIde":
          this.handleGlueIde(row);
          break;
        case "handleDelete":
          this.handleDelete(row);
          break;
        default:
          break;
      }
    },
    handleAdd() {
      this.$refs["editRef"].handleEdit();
    },
    handleExec(row) {
      this.$refs["execRef"].handleEdit(row);
    },
    handleGlueIde(row) {
      this.$refs["glueIdeRef"].handleEdit(row);
    },
    handleLog(row) {
      console.log('handleLog');
      this.$router.push('/joblog?jobGroup='+row.jobGroup+'&jobId=' + row.id);
    },
    handleUpdate(row) {
      this.$refs["editRef"].handleEdit(row);
    },
    handleCopy(row) {
      row.id = undefined;
      this.$refs["editRef"].handleEdit(row);
    },
    handleReg(row) {
      this.$refs["regRef"].handleEdit(row);
    },
    handleNexttime(row) {
      this.$refs["nextTiggerTimeRef"].handleEdit(row);
    },
    /** 删除按钮操作 */
    handleDelete(row) {
      this.$modal.confirm('是否确认删除:"' + row.jobDesc + '"？').then(() => {
        jobinfoRemove({id: row.id}).then(res => {
          this.getList();
          this.$modal.msgSuccess("删除成功");
        })
      }).catch(() => {});
    },
  }
};
</script>

<style lang="scss" scoped>
.el-dropdown {
  vertical-align: middle;
}
</style>
