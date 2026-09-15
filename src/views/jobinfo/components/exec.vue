<template>
  <el-dialog :title="title" v-model="open" width="600px">
    <el-form ref="editRef" :model="form" :rules="rules" label-width="120px">

      <el-form-item label="任务参数" prop="executorParam">
        <el-input v-model="form.executorParam" type="textarea" placeholder="请输入任务参数" />
      </el-form-item>

      <el-form-item label="机器地址" prop="addressList">
        <el-input v-model="form.addressList" type="textarea" placeholder="请输入本次执行的机器地址，为空则从执行器获取" />
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
import {jobinfoTrigger} from "@/api/jobinfo";

export default {
  name: "jobTrigger",
  emits: ['change'],
  data() {
    return {
      open: false,
      title: "",
      form: {
        executorParam: '',
        addressList: '',
      },
      rules: {
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
      this.form = JSON.parse(JSON.stringify(row));
      this.open = true;
      this.title = "执行一次";
    },
    /** 提交按钮 */
    submitForm() {
      this.$refs["editRef"].validate(valid => {
        if (valid) {
          jobinfoTrigger({
            id: this.form.id,
            executorParam: this.form.executorParam || '',
            addressList: this.form.addressList || ''
          }).then(res => {
            this.$modal.msgSuccess('执行一次成功');
            this.open = false;
          });
        }
      });
    },
  }
};
</script>
