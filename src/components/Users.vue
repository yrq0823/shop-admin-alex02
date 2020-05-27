<template>
  <div id="users">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索 -->
    <el-input class="input-with-select" placeholder="请输入搜索内容" v-model="query">
      <el-button @click="searchUser" slot="append" icon="el-icon-search"></el-button>
    </el-input>
    <el-button class="addBtn" type="success" plain @click="showAddUserDialog">添加用户</el-button>
    <!-- 表格 -->
    <el-table :data="userform">
      <el-table-column prop="username" label="姓名"></el-table-column>
      <el-table-column prop="email" label="邮箱"></el-table-column>
      <el-table-column prop="mobile" label="电话"></el-table-column>
      <el-table-column label="用户状态">
        <template slot-scope="{ row }">
          <el-switch
            @change="changeState(row)"
            v-model="row.mg_state"
            active-color="#13ce66"
            inactive-color="#ff4949"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template v-slot:default="{ row }">
          <el-button @click='showEditUserDialog(row)' icon="el-icon-edit" plain size="mini" type="primary"></el-button>
          <el-button @click="delUser(row.id)" icon="el-icon-delete" plain size="mini" type="danger"></el-button>
          <el-button icon="el-icon-check" plain size="mini" type="success">分配角色</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination
      background
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagenum"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    ></el-pagination>
    <!-- 添加用户的 Dialog -->
    <el-dialog @close='closeDialog' title="添加用户" :visible.sync="addUserDialogVisible" width="40%">
      <el-form
        status-icon
        :model="addUserForm"
        :rules="rules"
        ref="addUserForm"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addUserForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input type='password' v-model="addUserForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input placeholder='请输入邮箱' v-model="addUserForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input placeholder='请输入手机号' v-model="addUserForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addUserDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click='addUser'>确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户的 Dialog -->
    <el-dialog title="修改用户" :visible.sync="editUserDialogVisible" width="40%">
      <el-form
        status-icon
        :model="editUserForm"
        :rules="rules"
        ref="editUserForm"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-form-item label="用户名">
          <el-tag type='info'>{{ editUserForm.username }}</el-tag>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editUserForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editUserForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editUserDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click='editUser(editUserForm.id)'>确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      query: '',
      pagenum: 1,
      pagesize: 2,
      userform: [],
      total: 0,
      addUserDialogVisible: false,
      editUserDialogVisible: false,
      addUserForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editUserForm: {
        id: '',
        mobile: '',
        email: '',
        username: ''
      },
      rules: {
        username: [
          { required: true, message: '请输入用户名', trigger: ['blur', 'change'] },
          { min: 3, max: 12, message: '用户名长度在 3 到 12 个字符', trigger: ['blur', 'change'] }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: ['blur', 'change'] },
          { min: 3, max: 12, message: '密码长度在 3 到 12 个字符', trigger: ['blur', 'change'] }
        ],
        email: [
          { type: 'email', message: '请输入正确的邮箱格式', trigger: ['blur', 'change'] }
        ],
        mobile: [
          { pattern: /^1[3456789]\d{9}$/, message: '请输入正确的手机号格式', trigger: ['blur', 'change'] }
        ]
      }
    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    async getUserList () {
      const { meta, data } = await this.$axios.get('users', {
        params: {
          query: this.query,
          pagenum: this.pagenum,
          pagesize: this.pagesize
        }
      })
      console.log(data)
      if (meta.status === 200) {
        this.userform = data.users
        this.total = data.total
        this.pagenum = data.pagenum
      } else {
        this.$message.error(meta.msg)
      }
    },
    handleSizeChange (val) {
      this.pagesize = val
      this.pagenum = 1
      this.getUserList()
      console.log(`每页 ${val} 条`)
    },
    handleCurrentChange (val) {
      this.pagenum = val
      this.getUserList()
      console.log(`当前页: ${val}`)
    },
    async delUser (id) {
      try {
        await this.$confirm('你确定要删除吗？', '温馨提示', {
          type: 'warning'
        })
        const { meta } = await this.$axios.delete(`users/${id}`)
        if (meta.status === 200) {
          this.$message.success(meta.msg)
          if (this.userform.length === 1 && this.pagenum > 1) {
            this.pagenum--
          }
          this.getUserList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
        this.$message.info('取消删除')
      }
    },
    searchUser () {
      this.pagenum = 1
      this.getUserList()
    },
    async changeState (row) {
      const { meta } = await this.$axios.put(`users/${row.id}/state/${row.mg_state}`)
      if (meta.status === 200) {
        this.$message.success(meta.msg)
      } else {
        this.$message.error(meta.msg)
      }
    },
    showAddUserDialog () {
      this.addUserDialogVisible = true
    },
    // 添加用户
    async addUser () {
      try {
        await this.$refs.addUserForm.validate()
        const { meta } = await this.$axios.post('users', this.addUserForm)
        if (meta.status === 201) {
          this.addUserDialogVisible = false
          this.$message.success(meta.msg)
          // 添加完成后重置到最后一页
          this.pagenum = Math.ceil(++this.total / this.pagesize)
          this.getUserList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    },
    // 关闭对话框清空内容
    closeDialog () {
      this.$refs.addUserForm.resetFields()
    },
    // 修改用户对话框显示
    async showEditUserDialog (row) {
      this.editUserDialogVisible = true
      this.editUserForm.username = row.username
      this.editUserForm.mobile = row.mobile
      this.editUserForm.email = row.email
      this.editUserForm.id = row.id
      // const { meta, data } = await this.$axios.get(`users/${row.id}`)
      // if (meta.status === 200) {
      //   this.editUserForm = data
      //   this.addUserForm = data
      // } else {
      //   this.$message.error(meta.msg)
      // }
    },
    async editUser () {
      try {
        await this.$refs.editUserForm.validate()
        const { id, email, mobile } = this.editUserForm
        const { meta } = await this.$axios.put(`users/${id}`, { mobile, email })
        if (meta.status === 200) {
          this.$message.success(meta.msg)
          this.editUserDialogVisible = false
          this.getUserList()
        }
      } catch (e) {
        console.log(e)
      }
    }
  }
}
</script>

<style lang='scss' scoped>
#users {
  .el-breadcrumb {
    border-bottom: 1px solid #ccc;
    height: 40px;
    line-height: 40px;
    font-size: 14px;
  }
  .input-with-select {
    width: 300px;
    margin: 10px 0;
  }
  .addBtn {
    margin-left: 20px;
  }
}
</style>
