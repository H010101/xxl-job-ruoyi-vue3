<template>
  <el-dialog :title="title" v-model="open" width="480px">
    <el-form ref="editRef" :model="form" :rules="rules" label-width="120px">
      <el-form-item label="新密码" prop="password">
        <el-input v-model="form.password" placeholder="请输入新密码" />
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
import {userUpdatePwd} from "@/api/user";
import {removeToken } from '@/utils/auth'
import {ElMessageBox} from "element-plus";

export default {
  name: "PasswordEdit",
  data() {
    return {
      open: false,
      title: "修改密码",
      form: {},
      rules: {
        password: [{ required: true, message: "密码 不能为空", trigger: "blur" }],
      }
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
    init() {
      this.reset();
      this.open = true;
    },

    /** 提交按钮 */
    submitForm() {
      this.$refs["editRef"].validate(valid => {
        if (valid) {
          userUpdatePwd(this.form).then(res => {
            removeToken();
            ElMessageBox.alert('密码修改成功，请重新登录!', '重新登录!', {  confirmButtonText: '确定', callback: (action) => location.href = '/'})
          });
        }
      });
    }
  }
};
</script>
