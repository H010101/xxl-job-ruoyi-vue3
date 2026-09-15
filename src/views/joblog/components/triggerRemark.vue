<template>
  <el-dialog :title="title" v-model="open" width="600px">
    <div v-html="form.triggerMsg"></div>
    <template #footer>
      <div class="dialog-footer">
        <el-button type="primary" @click="cancel">确 定</el-button>
      </div>
    </template>
  </el-dialog>
</template>

<script>
export default {
  name: "TriggerRemark",
  data() {
    return {
      open: false,
      title: "执行备注",
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
      this.open = true;
    }
  }
};
</script>
