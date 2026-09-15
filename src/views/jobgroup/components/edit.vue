<template>
  <el-dialog :title="title" v-model="open" width="600px">
    <el-form ref="editRef" :model="form" :rules="rules" label-width="120px">

      <el-form-item label="AppName" prop="appname">
        <el-input v-model="form.appname" placeholder="请输入 AppName" />
      </el-form-item>
      <el-form-item label="名称" prop="title">
        <el-input v-model="form.title" placeholder="请输入 名称" />
      </el-form-item>

      <el-form-item label="注册方式" prop="addressType">
        <el-radio-group v-model="form.addressType">
          <el-radio v-for="item in AddressType" :key="item.value" :value="item.value">{{ item.label }}</el-radio>
        </el-radio-group>
      </el-form-item>

      <el-form-item label="机器地址" prop="addressList">
        <el-input v-model="form.addressList" type="textarea" :disabled="form.addressType === 0" placeholder="请输入执行器地址列表，多地址逗号分隔" />
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
import {jobgroupSave, jobgroupUpdate} from "@/api/jobgroup";
import AddressType from "@/api/dict/AddressType.json"

export default {
  name: "jobgroupEdit",
  emits: ['change'],
  data() {
    return {
      open: false,
      title: "",
      form: {},
      rules: {
        appname: [{ required: true, message: "AppName 不能为空", trigger: "blur" }],
        title: [{ required: true, message: "名称 不能为空", trigger: "blur" }],
        addressType: [{ required: true, message: "注册方式 不能为空", trigger: "blur" }],
        addressList: [{ required: false, message: "机器地址 不能为空", trigger: "blur" }],
      },
      AddressType,
    };
  },
  methods: {
    /** 表单重置 */
    reset() {
      this.form = {
        addressType: 0
      };
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
        this.open = true;
        this.title = "添加";
      } else {
        this.form = JSON.parse(JSON.stringify(row));
        this.open = true;
        this.title = "修改";
      }
    },
    /** 提交按钮 */
    submitForm() {
      this.$refs["editRef"].validate(valid => {
        if (valid) {
          if (this.form.addressType === 1 && !this.form.addressList) {
            this.$modal.msgError('手动录入地址，地址不能为空');
            return;
          }
          this.form.registryList = undefined;
          this.form.updateTime = undefined;
          if (this.form.id) {
            jobgroupUpdate(this.form).then(res => {
              this.$modal.msgSuccess('修改成功');
              this.open = false;
              this.$emit("change", true);
            });
          } else {
            jobgroupSave(this.form).then(res => {
              this.$modal.msgSuccess('新增成功');
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
