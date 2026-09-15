<template>
  <el-dialog :title="title" v-model="open" width="300px">
    <ol>
      <li v-for="item in nexts"><el-tag>{{item}}</el-tag></li>
    </ol>
    <template #footer>
      <div class="dialog-footer">
        <el-button type="primary" @click="cancel">确 定</el-button>
      </div>
    </template>
  </el-dialog>
</template>

<script>
import {jobinfoNextTriggerTime} from "@/api/jobinfo";

export default {
  name: "jobNextTiggerTime",
  data() {
    return {
      open: false,
      title: "",
      nexts: [],
    };
  },
  methods: {
    /** 表单重置 */
    reset() {
      this.nexts = [];
    },
    /** 取消按钮 */
    cancel() {
      this.open = false;
      this.reset();
    },
    // 新增/修改按钮操作
    handleEdit(row) {
      this.reset();
      jobinfoNextTriggerTime({
        scheduleType: row.scheduleType,
        scheduleConf: row.scheduleConf,
      }).then(res => {
        this.open = true;
        this.title = "下次执行时间";
        this.nexts = res.content;
      })
    },
  }
};
</script>
