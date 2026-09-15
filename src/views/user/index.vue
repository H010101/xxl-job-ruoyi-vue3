<template>
   <div class="app-container">
     <el-form :model="queryParams" ref="queryRef" :inline="true" label-width="68px">
        <el-form-item label="角色" prop="role">
          <el-select v-model="queryParams.role" placeholder="角色" filterable style='width: 120px' @change="handleQuery">
            <el-option v-for="dict in Role" :key="dict.value" :label="dict.label" :value="dict.value"/>
          </el-select>
        </el-form-item>
         <el-form-item label="账号" prop="username">
            <el-input v-model="queryParams.username" placeholder="账号" clearable style="width: 160px" @keyup.enter="handleQuery"/>
         </el-form-item>
         <el-form-item>
            <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
            <el-button icon="Refresh" @click="resetQuery">重置</el-button>
         </el-form-item>
        <el-form-item style="float: right;">
          <el-button type="primary" plain icon="Plus" @click="handleAdd">新增</el-button>
        </el-form-item>
      </el-form>
      <el-table v-loading="loading" :data="dataList">
        <el-table-column label="ID" prop="id" width="80"/>
        <el-table-column label="账号" prop="username" min-width="200"/>
        <el-table-column label="角色" prop="role" min-width="200">
          <template #default="{row}"><dict-tag :options="Role" :value="row.role"/></template>
        </el-table-column>
        <el-table-column label="操作" fixed='right' width="160">
          <template #default="{row}">
            <el-button link type="primary" icon="Edit" @click="handleUpdate(row)">编辑</el-button>
            <el-button link type="danger" icon="Delete" @click="handleDelete(row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>

     <pagination
         v-show="total > 0"
         :total="total"
         v-model:page="queryParams.current"
         v-model:limit="queryParams.size"
         @pagination="getList"
     />

     <edit ref="editRef" @change="getList"/>
   </div>
</template>

<script>
import {userPage, userRemove} from "@/api/user";
import Edit from "./components/edit"

import Role from "@/api/dict/Role.json"

export default {
  name: "User",
  components: {
    Edit
  },
  data() {
    return {
      Role,
      dataList: [],
      loading: false,
      total: 0,
      queryParams: {
        start: 0,
        length: 10,
        current: 1,
        size: 10,
        role: -1,
        username: '',
      }
    };
  },
  created() {
    this.getList();
  },
  methods: {
    /** 查询参数列表 */
    getList() {
      this.loading = true;
      this.queryParams.start = (this.queryParams.current - 1) *  this.queryParams.size
      userPage(this.queryParams).then(res => {
        this.dataList = res.data;
        // res.recordsFiltered
        this.total = res.recordsTotal;
      }).finally(() => {
        this.loading = false;
      });
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParams.current = 1;
      this.getList();
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.resetForm("queryRef");
      this.handleQuery();
    },
    handleAdd() {
      this.$refs["editRef"].handleEdit();
    },
    handleUpdate(row) {
      this.$refs["editRef"].handleEdit(row);
    },
    /** 删除按钮操作 */
    handleDelete(row) {
      this.$modal.confirm('是否确认删除:"' + row.username + '"？').then(() => {
        userRemove({id: row.id}).then(res => {
          this.getList();
          this.$modal.msgSuccess("删除成功");
        })
      }).catch(() => {});
    }
  }
};
</script>
