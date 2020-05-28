<template>
  <div id="category">
    <!-- 添加分类 -->
    <el-button @click='showAddCategoryDialog' type='success' plain class="addCategory">添加分类</el-button>
    <!-- 表格 -->
    <el-table
      v-loading="loading"
      element-loading-text="拼命加载中"
      element-loading-spinner="el-icon-loading"
      element-loading-background="rgba(255, 255, 255, 0.8)"
      :data='categoryList'
      row-key="cat_id"
      >
      <el-table-column prop="cat_name" label="分类名称"></el-table-column>
      <el-table-column label="是否有效">
        <template v-slot:default="{ row }">
          {{ row.cat_deleted ? '否' : '是' }}
        </template>
      </el-table-column>
      <el-table-column prop="cat_level" label="排序"></el-table-column>
      <el-table-column label="操作">
        <template>
          <el-button type='primary' plain icon='el-icon-edit' size='mini'></el-button>
          <el-button type='danger' plain icon='el-icon-delete' size='mini'></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination
      background
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagenum"
      :page-sizes="[5, 10, 15, 20]"
      :page-size="pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    <!-- 添加分类 dialog -->
    <el-dialog
      @close='closeDialog'
      title="添加分类"
      :visible.sync="addCategoryDialogVisible"
      width="40%"
      >
      <el-form status-icon :model="addCateForm" :rules="rules" ref="addCateForm" label-width="100px" class="demo-ruleForm">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name" placeholder='请输入分类名称'></el-input>
        </el-form-item>
         <el-form-item label="父级名称" prop='cat_pid'>
            <el-cascader
              :options="options"
              v-model='addCateForm.cat_pid'
              :props="props"
              clearable>
            </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCategoryDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click='addCategory'>确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  data () {
    return {
      categoryList: [],
      loading: true,
      pagenum: 1,
      pagesize: 5,
      total: 0,
      addCategoryDialogVisible: false,
      rules: {
        cat_name: [
          { required: true, message: '分类名称不能为空', trigger: ['blue', 'change'] }
        ]
      },
      addCateForm: {
        cat_name: '',
        cat_pid: [],
        cat_level: ''
      },
      options: [],
      props: {
        value: 'cat_id',
        label: 'cat_name',
        chidlren: 'children',
        checkStrictly: true
      }
    }
  },
  created () {
    this.getCategoryList()
  },
  methods: {
    async getCategoryList () {
      this.loading = true
      const { meta, data } = await this.$axios.get('categories', {
        params: {
          type: 3,
          pagenum: this.pagenum,
          pagesize: this.pagesize
        }
      })
      console.log('getCategoryList -> data', data)

      if (meta.status === 200) {
        this.categoryList = data.result
        this.total = data.total
      } else {
        this.$message.error(meta.msg)
      }
      this.loading = false
    },
    handleSizeChange (val) {
      this.pagesize = val
      this.pagenum = 1
      this.getCategoryList()
    },
    handleCurrentChange (val) {
      this.pagenum = val
      this.getCategoryList()
    },
    async showAddCategoryDialog () {
      this.addCategoryDialogVisible = true
      const { meta, data } = await this.$axios.get('categories', {
        params: {
          type: 2
        }
      })
      if (meta.status === 200) {
        this.options = data
      } else {
        this.$message.error(meta.msg)
      }
    },
    async addCategory () {
      try {
        await this.$refs.addCateForm.validate()
        const { meta } = await this.$axios.post('categories', {
          cat_pid: this.addCateForm.cat_pid[this.addCateForm.length - 1] || 0, // 0 表示没有父级
          cat_name: this.addCateForm.cat_name,
          cat_level: this.addCateForm.cat_pid.length
        })
        if (meta.status === 201) {
          this.$message.success(meta.msg)
          this.addCategoryDialogVisible = false
          this.getCategoryList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    },
    closeDialog () {
      this.$refs.addCateForm.resetFields()
    }
  }

}
</script>

<style lang='scss' scoped>
  #category {
    .el-table {
      margin-top: 10px;
    }
  }
</style>
