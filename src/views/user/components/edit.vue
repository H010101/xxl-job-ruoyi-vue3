<template>
  <el-dialog :title="title" v-model="open" width="600px">
    <el-form ref="editRef" :model="form" :rules="rules" label-width="120px">

      <el-form-item label="用户名" prop="username">
        <el-input v-model="form.username" :disabled="!!form.id" placeholder="请输入 用户名" />
      </el-form-item>

      <el-form-item label="密码" prop="password" v-if="!form.id">
        <el-input v-model="form.password" placeholder="请输入新密码" />
      </el-form-item>
      <el-form-item label="密码" v-if="!!form.id">
        <el-input v-model="form.password" placeholder="请输入新密码，为空则不更新密码" />
      </el-form-item>

      <el-form-item label="角色" prop="role">
        <el-radio-group v-model="form.role">
          <el-radio :value="0">普通用户</el-radio>
          <el-radio :value="1">管理员</el-radio>
        </el-radio-group>
      </el-form-item>

      <el-form-item label="权限" prop="permission">
        <el-input v-model="form.permission" type="textarea" placeholder="请输入 权限" />
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
import {userAdd, userUpdate} from "@/api/user";

export default {
  name: "UserEdit",
  emits: ['change'],
  data() {
    return {
      open: false,
      title: "",
      form: {},
      rules: {
        username: [{ required: true, message: "用户名不能为空", trigger: "blur" }],
        password: [{ required: true, message: "密码不能为空", trigger: "blur" }],
        role: [{ required: true, message: "角色 不能为空", trigger: "blur" }],
        permission: [{ required: false, message: "权限 不能为空", trigger: "blur" }],
      }
    };
  },
  methods: {
    /** 表单重置 */
    reset() {
      this.form = {
        role: 0
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
          if (this.form.id) {
            userUpdate(this.form).then(res => {
              this.$modal.msgSuccess('修改成功');
              this.open = false;
              this.$emit("change", true);
            });
          } else {
            userAdd(this.form).then(res => {
              this.$modal.msgSuccess('新增成功');
              this.open = false;
              this.$emit("change", true);
            });
          }
        }
      });
    }
  }
};
</script>
