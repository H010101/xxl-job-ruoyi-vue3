<template>
  <el-dialog :title="title" v-model="open" width="300px">
    <ol>
      <li v-for="item in jobgroup?.registryList"><el-tag>{{item}}</el-tag></li>
    </ol>
    <template #footer>
      <div class="dialog-footer">
        <el-button type="primary" @click="cancel">确 定</el-button>
      </div>
    </template>
  </el-dialog>
</template>

<script>
import {jobgroupLoadById} from "@/api/jobgroup";

export default {
  name: "jobReg",
  data() {
    return {
      open: false,
      title: "",
      jobgroup: {},
    };
  },
  methods: {
    /** 表单重置 */
    reset() {
      this.jobgroup = {};
    },
    /** 取消按钮 */
    cancel() {
      this.open = false;
      this.reset();
    },
    // 新增/修改按钮操作
    handleEdit(row) {
      this.reset();
      jobgroupLoadById({id: row.jobGroup}).then(res => {
        this.open = true;
        this.title = "注册节点";
        this.jobgroup = res.content;
      })
    },
  }
};
</script>
