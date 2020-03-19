<template>
    <div>
  <el-breadcrumb separator-class="el-icon-arrow-right">
  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
  <el-breadcrumb-item>权限管理</el-breadcrumb-item>
  <el-breadcrumb-item>角色列表</el-breadcrumb-item>
  </el-breadcrumb>
  <!-- 卡片视图 -->
    <el-card>
        <el-row>
            <el-col>
                 <el-button type="primary">添加角色</el-button>
            </el-col>
        </el-row>
        <!-- 角色列表区 -->
     <el-table :data="rolesList" border stripe>
    <el-table-column type="expand" label="#">
      <template slot-scope="scope">
        <el-row :class="['bdbottom', i1===0? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
          <!-- 一级权限 -->
          <el-col :span="5">
            <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
            <i class="el-icon-caret-right"></i>
          </el-col>
          <!-- 二级三级权限 -->
          <el-col :span="19">
            <!--通过for循环渲染二级菜单 -->
            <el-row :class="[i2===0? '' : 'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
              <el-col :span="6">
                 <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                  <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="18">
                  <el-tag type="warning" closable @close="removeRightById(scope.row, item3.id)" v-for="(item3) in item2.children" :key="item3.id">{{item3.authName}}</el-tag>
              </el-col>
            </el-row>
          </el-col>
        </el-row>
      </template>
    </el-table-column>
    <el-table-column type="index" label="#"></el-table-column>
    <el-table-column label="角色名称" prop="roleName"></el-table-column>
    <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
    <el-table-column label="操作" width="300px">
      <template slot-scope="scope">
          <el-button type="primary"  icon="el-icon-edit" size="mini">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
          <el-button type="warning" icon="el-icon-setting" size="mini" @click="setRightDialog(scope.row)">分配权限</el-button>
      </template>
    </el-table-column>
    </el-table>
    </el-card>
    <!-- 分配权限对话框 -->
    <el-dialog
     title="提示"
     :visible.sync="setRightDialogVisible"
     width="50%" @close="clearDialog">
     <el-tree :data="rightList" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defkeys" ref="treeRef"></el-tree>
  <span slot="footer" class="dialog-footer">
    <el-button @click="setRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </span>
</el-dialog>
    </div>
</template>
<script>
export default {
  data () {
    return {
      rolesList: [],
      setRightDialogVisible: false,
      rightList: [],
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      defkeys: [],
      roleId: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolesList = res.data
    },
    async removeRightById (role, RightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除操作')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${RightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }
      // this.getRolesList()
      role.children = res.data
    },
    async setRightDialog (role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败')
      }
      this.rightList = res.data
      this.getLeafKeys(role, this.defkeys)
      this.setRightDialogVisible = true
    },
    // 通过递归的方式获取角色的三级权限的id
    getLeafKeys (role, arr) {
      const firstRights = role.children
      for (var i = 0; i < firstRights.length; i++) {
        var secondRights = firstRights[i].children
        for (var j = 0; j < secondRights.length; j++) {
          var thirdRights = secondRights[j].children
          for (var k = 0; k < thirdRights.length; k++) {
            arr.push(thirdRights[k].id)
          }
        }
      }
    },
    clearDialog () {
      this.defkeys = []
    },
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const IdStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: IdStr })
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败')
      }
      this.$message.success('分配权限成功')
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
  border: 1px solid #f60;
}
.bdbottom {
  border: 1px solid #f60;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
