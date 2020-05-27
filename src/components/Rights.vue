<template>
  <div id="rights">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 权限列表 -->
    <el-table
      ref="singleTable"
      :data="tableData"
      highlight-current-row
      style="width: 100%"
    >
      <el-table-column type="index" label='#'></el-table-column>
      <el-table-column property="authName" label="权限名称" ></el-table-column>
      <el-table-column property="path" label="路径" ></el-table-column>
      <el-table-column label="层级">
        <template v-slot:default="{ row }">
          <p v-if="row.level === '0'">一级</p>
          <p v-if="row.level === '1'">二级</p>
          <p v-if="row.level === '2'">三级</p>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  data () {
    return {
      tableData: []
    }
  },
  created () {
    this.getRightsList()
  },
  methods: {
    async getRightsList () {
      const { meta, data } = await this.$axios.get('rights/list')
      if (meta.status === 200) {
        this.tableData = data
      } else {
        this.$message.error(meta.msg)
      }
    }
  }
}
</script>

<style scoped lang='scss'>
#rights {
  .el-breadcrumb {
    border-bottom: 1px solid #ccc;
    height: 40px;
    line-height: 40px;
    font-size: 14px;
  }
  .el-table {
    margin-top: 10px;
  }
}
</style>
