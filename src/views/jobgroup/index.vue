<template>
   <div class="app-container">
     <el-form :model="queryParams" ref="queryRef" :inline="true" label-width="80px">
       <el-form-item label="AppName" prop="appname">
         <el-input v-model="queryParams.appname" placeholder="AppName" clearable style="width: 160px" @keyup.enter="handleQuery"/>
       </el-form-item>
       <el-form-item label="名称" prop="title">
         <el-input v-model="queryParams.title" placeholder="名称" clearable style="width: 160px" @keyup.enter="handleQuery"/>
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
        <el-table-column label="AppName" prop="appname" min-width="200"/>
        <el-table-column label="名称" prop="title" min-width="120"/>
        <el-table-column label="注册方式" prop="addressType" min-width="100">
          <template #default="{row}"><dict-tag :options="AddressType" :value="row.addressType"/></template>
        </el-table-column>
        <el-table-column label="OnLine 机器地址" prop="registryList" min-width="200">
          <template #default="{row}">
            <div>
              <el-tag v-for="item in row.registryList">{{item}}</el-tag>
            </div>
          </template>
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
import {jobgroupPage, jobgroupRemove} from "@/api/jobgroup";
import Edit from "./components/edit"
import AddressType from "@/api/dict/AddressType.json"

export default {
  name: "Jobgroup",
  components: { Edit },
  data() {
    return {
      dataList: [],
      loading: false,
      total: 0,
      queryParams: {
        start: 0,
        length: 10,
        current: 1,
        size: 10,
        appname: '',
        title: '',
      },
      AddressType,
    };
  },
  created() {
    this.getList();
  },
  methods: {
    /** 查询参数列表 */
    getList() {
      this.loading = true;
      this.queryParams.start = (this.queryParams.current - 1) * this.queryParams.size;
      jobgroupPage(this.queryParams).then(res => {
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
      this.$modal.confirm('是否确认删除:"' + row.appname + '"？').then(() => {
        jobgroupRemove({id: row.id}).then(res => {
          this.getList();
          this.$modal.msgSuccess("删除成功");
        })
      }).catch(() => {});
    },
  }
};
</script>
