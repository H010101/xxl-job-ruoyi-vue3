<template>
  <el-dialog v-model="open" width="80%" @close="cancel">
    <template #header="{ close, titleId, titleClass }">
      <div class="my-header">
        <div :id="titleId" :class="titleClass">执行日志</div>
        <div>
          每&nbsp;<el-input-number v-model="refreshTime" style="width: 120px" :min="2" placeholder="定时" clearable />&nbsp;秒
          <el-button type="primary" @click="getLog">刷新一次</el-button>
          <span style="padding: 0 12px 0 12px">最后刷新时间: {{parseTime(logTime)}}</span>
        </div>
      </div>
    </template>

    <monaco-editor v-model="form.logContent" language="html" height="660px"/>
    <template #footer>
      <div class="dialog-footer">
        <el-button type="primary" @click="cancel">确 定</el-button>
      </div>
    </template>
  </el-dialog>
</template>

<script>
import {joblogLogDetailCat} from "@/api/joblog";
import MonacoEditor from '@/components/MonacoEditor'

export default {
  name: "LogDetail",
  components: {
    MonacoEditor
  },
  data() {
    return {
      open: false,
      title: "",
      baseTime: 1000,
      refreshTime: 5,
      interval: null,
      logTime: null,
      param: {},
      form: {}
    };
  },
  methods: {
    /** 表单重置 */
    reset() {
      this.form = {};
      this.param = {};
      this.resetForm("editRef");
    },

    /** 取消按钮 */
    cancel() {
      this.open = false;
      this.reset();
    },

    // 新增/修改按钮操作
    init(row) {
      this.reset();
      this.param = row;
      this.getLog();
      this.setInterval4Log();
    },

    setInterval4Log() {
      if (this.interval !== null) {
        return;
      }
      this.interval = setInterval(() => {
        if (this.refreshTime && this.baseTime % (this.refreshTime * 1000) === 0) {
          this.getLog();
        }
        this.baseTime = this.baseTime + 1000;
      }, 1000)
    },

    getLog() {
      if (!this.param.id) {
        clearInterval(this.interval);
        return;
      }
      joblogLogDetailCat({
        logId: this.param.id,
        fromLineNum: 1
      }).then(res => {
        this.form = res.content;
        if (this.refreshTime === 0) {
          this.$modal.msgSuccess("获取日志成功！");
        }
        this.logTime = new Date();
        this.open = true;
      })
    }
  }
};
</script>
<style scoped>
.my-header {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  gap: 16px;
}
</style>
