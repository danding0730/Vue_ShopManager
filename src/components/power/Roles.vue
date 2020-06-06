<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator=">">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 添加角色按钮区 -->
      <el-row>
        <el-col>
          <el-button type="primary" round @click="addRoleDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data="rolesList" border stripe>
        <!-- 展开详细权限区域 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom', id1 === 0 ? 'bdtop' : '','vcenter']"
              v-for="(item1, id1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <!-- 通过for循环嵌套渲染二级权限 -->
                <el-row
                  :class="[id2 === 0 ? '' : 'bdtop','vcenter']"
                  v-for="(item2, id2) in item1.children"
                  :key="item2.id"
                >
                  <!-- 二级权限渲染 -->
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                    >{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 三级权限渲染 -->
                  <el-col :span="18">
                    <!-- <el-row
                      :class="[i3 === 0 ? '' : 'bdtop']"
                      v-for="(item3, i3) in item2.children"
                      :key="item3"
                    >
                      <el-col :span="18">
                        <el-tag type="warning">{{item3.authName}}</el-tag>
                      </el-col>
                    </el-row>-->
                    <el-tag
                      v-for="(item3) in item2.children"
                      :key="item3.id"
                      type="warning"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 渲染数据和操作区域 -->
        <el-table-column type="index" label="序号"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <!-- 右侧操作按钮区域 -->
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              round
              size="mini"
              @click="showEditRoleDialog(scope.row.id)"
            >编辑</el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              round
              size="mini"
              @click="deleteRole(scope.row.id)"
            >删除</el-button>
            <el-button
              type="warning"
              icon="el-icon-setting"
              round
              size="mini"
              @click="showSetRightDialog(scope.row)"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 操作弹出的对话框区域 -->

    <!-- 添加角色内容区域 -->
    <el-dialog title="添加角色" :visible.sync="addRoleDialogVisible" width="50%" @close="addRoleClosed">
      <el-form
        :model="addRoleForm"
        :rules="addRoleRules"
        ref="addRoleRef"
        label-width="70px"
        label-position="top"
      >
        <el-form-item label="用户名称" prop="roleName">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRoleForm.roleDesc" type="textarea"></el-input>
        </el-form-item>
      </el-form>
      <!-- 添加角色对话框的底部按钮区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑用户信息区域 -->
    <el-dialog
      title="修改用户信息"
      :visible.sync="editRoleDialogVisible"
      width="50%"
      @close="editRoleClosed"
    >
      <el-form
        :model="editRoleForm"
        :rules="editRoleRules"
        ref="editRoleRef"
        label-width="70px"
        label-position="top"
      >
        <el-form-item label="角色名称">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="email">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 编辑用户信息的底部按钮区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo(editRoleForm.roleId)">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限区域 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed"
    >
      <!-- 权限树形控件区域 -->
      <el-tree
        :data="setRightTree"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rolesList: [],
      addRoleForm: {
        roleName: ' ',
        roleDesc: ' '
      },
      editRoleForm: {},
      // 控制各对话框
      addRoleDialogVisible: false,
      editRoleDialogVisible: false,
      setRightDialogVisible: false,
      setRightTree: {},
      // 权限树形控件绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 树形控件默认勾选的值 用户已有的权限
      defKeys: [],
      // 正在分配权限的角色ID
      roleId: '',
      addRoleRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请添加角色描述', trigger: 'blur' }
        ]
      },
      editRoleRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败！')
      }
      this.rolesList = res.data
    },
    addRoleClosed() {
      this.$refs.addRoleRef.resetFields()
    },
    editRoleClosed() {
      this.$refs.editRoleRef.resetFields()
    },
    addRole() {
      this.$refs.addRoleRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addRoleForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败，请重试！')
        }
        this.$message.success('添加用户成功！')
        this.addRoleDialogVisible = false
        this.getRolesList()
      })
    },
    async showEditRoleDialog(id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('获取编辑列表失败！')
      }
      this.editRoleForm = res.data
      this.editRoleDialogVisible = true
    },
    editRoleInfo(id) {
      this.$refs.editRoleRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          'roles/' + this.editRoleForm.roleId,
          {
            roleName: this.editRoleForm.roleName,
            roleDesc: this.editRoleForm.roleDesc
          }
        )
        if (res.meta.status !== 200) return this.$message.error('修改失败！')
        this.editRoleDialogVisible = false
        this.$message.success('修改成功！')
        this.getRolesList()
      })
    },
    async deleteRole(id) {
      const confirmMsg = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmMsg !== 'confirm') {
        return this.$message.info('已取消操作')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('操作失败，请重试！')
      }
      this.$message.success('删除成功！')
      this.getRolesList()
    },
    async removeRightById(role, rightId) {
      const confirmMsg = await this.$confirm(
        '此操作将永久删除该权限, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmMsg !== 'confirm') {
        return this.$message.info('已取消操作')
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}/`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('操作失败，请重试！')
      }
      this.$message.success('删除权限成功！')
      role.children = res.data
    },
    async showSetRightDialog(role) {
      this.roleId = role.id
      // 获取权限数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败！')
      }
      // 把数据保存到data中的 setRightTree 中
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
      this.setRightTree = res.data
    },
    // 通过递归获取树形控件下三级权限的ID，并且保存到defKeys中，渲染到页面上
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    setRightDialogClosed() {
      this.defKeys = []
    },
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')

      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('操作失败，请重试！')
      }
      this.$message.success('操作成功，成功设置了角色权限')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
