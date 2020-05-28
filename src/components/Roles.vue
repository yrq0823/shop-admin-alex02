<template>
  <div id="roles">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 添加角色按钮 -->
    <el-button type='success' plain @click='showAddRole'>添加角色</el-button>
    <!-- 角色列表 -->
    <el-table :data='roleform'>
      <el-table-column type='expand'>
        <template v-slot:default='{ row }'>
          <p v-show='row.children.length === 0'>暂无权限</p>
          <el-row class="level1"  v-for='level1 in row.children' :key='level1.id'>
            <el-col :span='4'><el-tag @close='delRights(row, level1.id)' closable>{{ level1.authName }}</el-tag><i class="el-icon-arrow-right"></i></el-col>
            <el-col :span='20'>
              <el-row class="level2" v-for='level2 in level1.children' :key='level2.id'>
                <el-col :span='4'><el-tag @close='delRights(row, level2.id)' closable type="success">{{ level2.authName }}</el-tag><i class="el-icon-arrow-right"></i></el-col>
                <el-col :span='16'>
                  <el-tag @close='delRights(row, level3.id)' class="level3" closable v-for='level3 in level2.children' :key='level3.id' type="warning">{{ level3.authName }}</el-tag>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
        </template>
      </el-table-column>
      <el-table-column type='index' label='#'></el-table-column>
      <el-table-column property='roleName' label='角色名称'></el-table-column>
      <el-table-column property='roleDesc' label='描述'></el-table-column>
      <el-table-column label='操作'>
        <template v-slot:default='{ row }'>
          <el-button plain size='mini' icon='el-icon-edit' type='primary' @click='showEditDialog(row)'></el-button>
          <el-button plain size='mini' icon='el-icon-delete' type='danger' @click='delRole(row.id)'></el-button>
          <el-button plain size='mini' icon='el-icon-check' type='success' @click='showAssignDialog(row)'>分配权限</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分配权限的 dialog -->
    <el-dialog
      title="分配权限"
      :visible.sync="assignDialogVisible"
      width="40%"
      >
      <el-tree
        node-key="id"
        default-expand-all
        :props='props'
        :data="data"
        show-checkbox
        ref="tree"
      >
      </el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="assignDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click='assignRights'>确 定</el-button>
      </span>
    </el-dialog>
    <!-- 添加角色的 dialog -->
    <el-dialog
      @close='closeDialog'
      title="添加角色"
      :visible.sync="addRoleDialogVisible"
      width="40%"
      >
      <el-form :model="roleForm" :rules="rules" ref="roleForm" label-width="100px" >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="roleForm.roleName" placeholder='请输入角色名称'></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="roleForm.roleDesc" placeholder='请输入角色描述'></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click='addRole'>确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑角色的 dialog -->
    <el-dialog
      title="修改角色"
      :visible.sync="editRoleDialogVisible"
      width="40%"
      >
      <el-form :model="editRoleForm" :rules="rules" ref="editRoleForm" label-width="100px" >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click='editRole'>确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      roleform: [],
      assignDialogVisible: false,
      addRoleDialogVisible: false,
      editRoleDialogVisible: false,
      data: [],
      props: {
        label: 'authName',
        children: 'children'
      },
      // 准备一个变量用于存储正在操作的角色 id,显示对话框的时候获取，分配角色的时候使用
      roleId: '',
      roleForm: {
        id: '',
        roleName: '',
        roleDesc: ''
      },
      editRoleForm: {
        id: '',
        roleName: '',
        roleDesc: ''
      },
      rules: {
        roleName: [
          { required: true, message: '角色名称不能为空', trigger: ['blur', 'change'] }
        ],
        roleDesc: [
          { required: true, message: '角色描述不能为空', trigger: ['blur', 'change'] }
        ]
      }
    }
  },
  created () {
    this.getRoleList()
  },
  methods: {
    async getRoleList () {
      const { meta, data } = await this.$axios.get('roles')
      if (meta.status === 200) {
        this.roleform = data
      } else {
        this.$message.error(meta.msg)
      }
    },
    async delRights (row, rightid) {
      const { meta, data } = await this.$axios.delete(`roles/${row.id}/rights/${rightid}`)
      if (meta.status === 200) {
        this.$message.success(meta.msg)
        row.children = data
      } else {
        this.$message.error(meta.msg)
      }
    },
    async showAssignDialog (row) {
      this.roleId = row.id
      this.assignDialogVisible = true
      console.log(row.id)
      const { data, meta } = await this.$axios.get('rights/tree')
      if (meta.status === 200) {
        this.data = data
        const ids = []
        // 获取所有三级权限的 id
        row.children.forEach(level1 => {
          level1.children.forEach(level2 => {
            level2.children.forEach(level3 => {
              ids.push(level3.id)
            })
          })
        })
        this.$refs.tree.setCheckedKeys(ids)
      } else {
        this.$message.error(meta.msg)
      }
    },
    async assignRights () {
      const id1 = this.$refs.tree.getCheckedKeys()
      const id2 = this.$refs.tree.getHalfCheckedKeys()
      const ids = [...id1, ...id2].join(',')
      const { meta } = await this.$axios.post(`roles/${this.roleId}/rights`, {
        rids: ids
      })
      if (meta.status === 200) {
        this.$message.success(meta.msg)
        this.assignDialogVisible = false
        this.getRoleList()
      } else {
        this.$message.error(meta.msg)
      }
    },
    // 删除角色
    async delRole (id) {
      try {
        await this.$confirm('你确定要删除吗？', '温馨提示', {
          type: 'warning'
        })
        const { meta } = await this.$axios.delete(`roles/${id}`)
        if (meta.status === 200) {
          this.$message.success(meta.msg)
          this.getRoleList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        this.$message.info('取消删除')
      }
    },
    // 添加角色对话框
    showAddRole () {
      this.addRoleDialogVisible = true
    },
    // 添加角色
    async addRole () {
      const { meta } = await this.$axios.post('roles', this.roleForm)
      if (meta.status === 201) {
        this.$message.success(meta.msg)
        this.addRoleDialogVisible = false
        this.getRoleList()
      } else {
        this.$message.error(meta.msg)
      }
    },
    // 编辑角色对话框
    showEditDialog (row) {
      console.log(row)
      this.editRoleDialogVisible = true
      this.editRoleForm.id = row.id
      this.editRoleForm.roleName = row.roleName
      this.editRoleForm.roleDesc = row.roleDesc
    },
    // 编辑角色
    async editRole () {
      try {
        await this.$refs.editRoleForm.validate()
        const { id, roleName, roleDesc } = this.editRoleForm
        const { meta } = await this.$axios.put(`roles/${id}`, {
          roleName,
          roleDesc
        })
        console.log('meta,', meta)
        if (meta.status === 200) {
          this.$message.success('操作成功')
          this.editRoleDialogVisible = false
          this.getRoleList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    },
    closeDialog () {
      this.$refs.roleForm.resetFields()
    }
  }
}
</script>

<style scoped lang='scss'>
  #roles {
    .el-table {
      margin-top: 10px;
      .level1 {
        margin-bottom: 10px;
        border-bottom: 1px solid #ccc;
      }

      .level2 {
        margin-bottom: 10px;
      }
      .level3 {
        margin-left: 5px;
        margin-bottom: 10px;
      }
    }
  }
</style>
