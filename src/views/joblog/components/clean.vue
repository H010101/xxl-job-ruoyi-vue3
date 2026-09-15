<template>
  <el-dialog :title="title" v-model="open" width="600px">
    <el-form ref="editRef" :model="form" label-width="120px">

      <el-form-item label="执行器" prop="jobGroupName">
        <el-input v-model="form.jobGroupName" disabled />
      </el-form-item>

      <el-form-item label="任务" prop="jobName">
        <el-input v-model="form.jobName" disabled />
      </el-form-item>



      <el-form-item label="清理方式" prop="type">
        <el-select v-model="form.type" placeholder="请选择 清理方式" filterable style='width: 660px'>
          <el-option v-for="dict in CleanType" :key="dict.value" :label="dict.label" :value="dict.value"/>
        </el-select>
      </el-form-item>

    </el-form>
    <template #footer>
      <div class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </template>
  </el-dialog>
</template>

<script>
import {joblogClearLog} from "@/api/joblog";

import CleanType from "@/api/dict/CleanType.json"

export default {
  name: "CleanLog",
  emits: ['change'],
  data() {
    return {
      CleanType,
      open: false,
      title: "日志清理",
      form: {}
    };
  },
  methods: {
    /** 表单重置 */
    reset() {
      this.form = {};
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
      this.form = JSON.parse(JSON.stringify(row));
      this.form.type = 1;
      this.open = true;
    },

    submitForm() {
      joblogClearLog({
        jobGroup: this.form.jobGroup,
        jobId: this.form.jobId,
        type: this.form.type
      }).then(res => {
        this.$modal.msgSuccess('清理成功');
        this.open = false;
        this.$emit("change", true);
      });
    }
  }
};
</script>
