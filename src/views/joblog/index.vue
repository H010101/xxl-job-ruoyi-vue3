<template>
   <div class="app-container">
     <el-form :model="queryParams" ref="queryRef" :inline="true">
       <el-form-item label="执行器" prop="jobGroup">
         <el-select v-model="queryParams.jobGroup" placeholder="执行器" filterable style='width: 200px' @change="changeJobGroup">
           <el-option v-for="item in appOptions" :key="item.id" :label="item.title" :value="item.id"/>
         </el-select>
       </el-form-item>
       <el-form-item label="任务" prop="jobId">
         <el-select v-model="queryParams.jobId" placeholder="任务" filterable style='width: 200px' @change="handleQuery">
           <el-option v-for="item in jobsOptions" :key="item.id" :label="item.jobDesc" :value="item.id"/>
         </el-select>
       </el-form-item>
       <el-form-item label="状态" prop="logStatus">
         <el-select v-model="queryParams.logStatus" placeholder="状态" filterable style='width: 100px' @change="handleQuery">
           <el-option v-for="item in LogStatus" :key="item.value" :label="item.label" :value="item.value"/>
         </el-select>
       </el-form-item>
       <el-form-item>
         <el-date-picker
             v-model="dateRange"
             style='width: 380px'
             :clearable="false"
             value-format="YYYY-MM-DD HH:mm:ss" type="datetimerange"
             range-separator="-"
             start-placeholder="开始时间" end-placeholder="结束时间"/>
       </el-form-item>
       <el-form-item>
         <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
         <el-button icon="Refresh" @click="resetQuery">重置</el-button>
       </el-form-item>
       <el-form-item style="float: right;">
         <el-button type="info" plain icon="Delete" @click="handleClean">清理</el-button>
       </el-form-item>
      </el-form>
      <el-table v-loading="loading" :data="dataList">
        <el-table-column label="ID" prop="id" width="80"/>
        <el-table-column label="任务ID" prop="jobId" min-width="160"/>
        <el-table-column label="调度时间" prop="triggerTime" min-width="160">
          <template #default="{row}"><span>{{ parseTime(row.triggerTime) }}</span></template>
        </el-table-column>
        <el-table-column label="调度结果" prop="triggerCode" min-width="100">
          <template #default="{row}"><dict-tag :options="TriggerResult" :value="row.triggerCode"/></template>
        </el-table-column>
        <el-table-column label="调度备注" prop="triggerMsg" min-width="100">
          <template #default="{row}">
            <el-button link type="primary" icon="View" @click="handleRriggerRemark(row)">查看</el-button>
          </template>
        </el-table-column>
        <el-table-column label="执行时间" prop="handleTime" min-width="160">
          <template #default="{row}"><span>{{ parseTime(row.handleTime) }}</span></template>
        </el-table-column>
        <el-table-column label="执行结果" prop="handleCode" min-width="100">
          <template #default="{row}"><dict-tag :options="HandleResult" :value="row.handleCode"/></template>
        </el-table-column>
        <el-table-column label="执行备注" prop="handleMsg" min-width="160">
          <template #default="{row}"><span>{{ row.handleMsg }}</span></template>
        </el-table-column>
        <el-table-column label="操作" fixed='right' width="160">
          <template #default="{row}">
            <el-button link type="primary" icon="View" @click="handleLogDetail(row)">执行日志</el-button>
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

     <clean ref="cleanRef" @change="cleanResult"/>
     <trigger-remark ref="triggerRemarkRef"/>
     <log-detail ref="logDetailRef"/>
   </div>
</template>

<script>
import {joblogPage, joblogGetJobsByGroup} from "@/api/joblog";
import {jobgroupPage} from "@/api/jobgroup";
import { parseTime } from '@/utils/ruoyi';
import LogStatus from "@/api/dict/LogStatus.json"
import TriggerResult from "@/api/dict/TriggerResult.json"
import HandleResult from "@/api/dict/HandleResult.json"

import Clean from "./components/clean"
import TriggerRemark from "./components/triggerRemark"
import LogDetail from "./components/logDetail"

export default {
  name: "Joblog",
  components: {
    Clean,
    TriggerRemark,
    LogDetail
  },
  data() {
    return {
      LogStatus,
      TriggerResult,
      HandleResult,
      appOptions: [],
      jobsOptions: [],
      dataList: [],
      loading: false,
      total: 0,
      dateRange: [],
      queryParams: {
        start: 0,
        length: 10,
        current: 1,
        size: 10,
        jobGroup: -1,
        jobId: 0,
        logStatus: -1,
        filterTime: '2023-08-12 00:00:00 - 2023-08-12 23:59:59'
      }
    };
  },
  created() {
    this.init();
  },
  methods: {
    parseTime,
    init() {
      const now = new Date();
      this.dateRange = [
        parseTime(now, '{y}-{m}-{d}') + ' 00:00:00',
        parseTime(now, '{y}-{m}-{d}') + ' 23:59:59'
      ];
      this.appOptions.push({id: -1, title: '全部'})
      this.jobsOptions.push({id: 0, jobDesc: '全部'})

      const {query} = this.$route;
      let jobGroup = query?.jobGroup;
      let jobId = query?.jobId;

      if (jobGroup && jobGroup > 0) {
        this.queryParams.jobGroup = Number(jobGroup);
        this.changeJobGroup(jobGroup);
      }
      if (jobId && jobId > 0) {
        this.queryParams.jobId = Number(jobId);
      }

      // 非跳转场景，进入页面查询一次
      if (!jobGroup && !jobId) {
        this.handleQuery();
      }
      this.getApps();
    },
    getApps() {
      jobgroupPage({
        start: 0,
        length: 10000,
        appname: '',
        title: ''
      }).then(res => {
        const data = res.data;
        if (data && data.length > 0) {
          for (const a of data) {
            this.appOptions.push(a);
          }
        }
      });
    },
    changeJobGroup(jobGroup) {
      this.handleQuery();
      if (jobGroup === -1) {
        return;
      }
      this.getJobsOptions();
    },
    getJobsOptions() {
      if (!this.queryParams.jobGroup) {
        return;
      }
      this.jobsOptions = [];
      this.jobsOptions.push({id: 0, jobDesc: '全部'})
      if (this.queryParams.jobGroup === -1) {
        return;
      }
      joblogGetJobsByGroup({jobGroup: this.queryParams.jobGroup}).then(res => {
        const data = res.content;
        if (data && data.length > 0) {
          for (let d of data) {
            this.jobsOptions.push(d)
          }
        }
      })
    },
    /** 查询参数列表 */
    getList() {
      this.loading = true;

      if (this.dateRange.length === 2) {
        const from = this.dateRange[0];
        const to = this.dateRange[1];
        this.queryParams.filterTime = from + ' - '+ to;
      }
      this.queryParams.start = (this.queryParams.current - 1) *  this.queryParams.size
      joblogPage(this.queryParams).then(res => {
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
    handleClean() {
      const row = {};
      row.jobGroup = this.queryParams.jobGroup;
      row.jobId = this.queryParams.jobId;
      for (const t of this.appOptions) {
        if (t.id === row.jobGroup) {
          row.jobGroupName = t.title;
          break;
        }
      }
      for (const t of this.jobsOptions) {
        if (t.id === row.jobId) {
          row.jobName = t.jobDesc;
          break;
        }
      }
      this.$refs["cleanRef"].init(row);
    },
    cleanResult() {
      this.getList();
    },
    handleRriggerRemark(row) {
      this.$refs["triggerRemarkRef"].init(row);
    },
    handleLogDetail(row) {
      this.$refs["logDetailRef"].init(row);
    }
  }
};
</script>
