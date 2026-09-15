<template>
  <el-dialog :title="title" v-model="open" width="1080px">
    <el-form ref="editRef" :model="form" :rules="rules" label-width="108px">
      <el-form-item label="任务参数" prop="glueSource">
        <monaco-editor v-model="form.glueSource" language="java" placeholder="请编辑 GLUE 源码" height="320px"/>
      </el-form-item>
      <el-form-item label="源码备注" prop="glueRemark">
        <el-input v-model="form.glueRemark" placeholder="请输入 GLUE 源码 修改备注" />
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
import {jobcodeSave} from "@/api/jobinfo";
import MonacoEditor from '@/components/MonacoEditor'

export default {
  name: "JobinfoGlue",
  components: { MonacoEditor },
  emits: ['change'],
  data() {
    return {
      open: false,
      title: "",
      form: {},
      rules: {
        glueSource: [{ required: true, message: "源代码 不能为空", trigger: "blur" }],
        glueRemark: [{ required: true, min: 4, max: 63, message: "源代码 备注不能少于4个字符！", trigger: "blur" }],
      },
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
    handleEdit(row) {
      this.reset();
      if (!row || !row.id) {
        return;
      }

      this.form = JSON.parse(JSON.stringify(row));
      this.open = true;
      this.title = "编辑: " + row.glueType + ": " + row.jobDesc;
    },
    /** 提交按钮 */
    submitForm() {
      this.$refs["editRef"].validate(valid => {
        if (valid) {
          if (this.form.id) {
            jobcodeSave({
              id: this.form.id,
              glueSource: this.form.glueSource,
              glueRemark: this.form.glueRemark
            }).then(res => {
              this.$modal.msgSuccess('修改成功');
              this.open = false;
              this.$emit("change", true);
            });
          }
        }
      });
    },
  }
};
</script>
