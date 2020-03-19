<template>
    <div>
  <el-breadcrumb separator-class="el-icon-arrow-right">
  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
  <el-breadcrumb-item>商品管理</el-breadcrumb-item>
  <el-breadcrumb-item>参数列表</el-breadcrumb-item>
  </el-breadcrumb>
  <!-- 卡片视图区 -->
  <el-card>
      <!-- 警告qu -->
       <el-alert title="注意：只允许为第三级分类设置参数" type="warning" show-icon></el-alert>
      <!-- 选择商品分类区 -->
       <el-row>
           <el-col>
               <span>选择商品分类区：</span>
               <!-- 级联选择区 -->
             <el-cascader
            expand-trigger="hover"
            v-model="selectedCateKeys"
            :options="cateList"
            :props="cateProps"
            @change="handleChange">
            </el-cascader>
           </el-col>
       </el-row>
       <!-- tab页区 -->
         <el-tabs v-model="activeName" @tab-click="handleTabClick">
         <el-tab-pane label="动态参数" name="many">
           <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加参数</el-button>
           <el-table :data="manyDataList" border stripe>
           <el-table-column type="expand">
             <template slot-scope="scope">
               <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="deletParams(i, scope.row)">
                 {{item}}
               </el-tag>
               <el-input
              class="input-new-tag"
              v-if="scope.row.inputVisible"
              v-model="scope.row.inputValue"
              ref="saveTagInput"
              size="small"
              @keyup.enter.native="handleInputConfirm(scope.row)"
              @blur="handleInputConfirm(scope.row)">
            </el-input>
            <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
             </template>
           </el-table-column>
           <el-table-column type="index"></el-table-column>
           <el-table-column label="参数名称" prop="attr_name"></el-table-column>
           <el-table-column label="操作">
             <template slot-scope="scope">
               <el-button size="mini" type="primary" icon="el-icon-edit"  @click="editShowDiaLog(scope.row.attr_id)">编辑</el-button>
               <el-button size="mini" type="danger" icon="el-icon-delete" @click="deleteParams(scope.row.attr_id)">删除</el-button>
             </template>
           </el-table-column>
           </el-table>
         </el-tab-pane>
         <el-tab-pane label="静态属性" name="only" :disabled="isBtnDisabled">
           <el-button type="primary" size="mini" @click="addDialogVisible=true">添加属性</el-button>
           <el-table :data="onlyDataList" border stripe>
              <el-table-column type="expand">
             <template slot-scope="scope">
               <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="deletParams(i, scope.row)">
                 {{item}}
               </el-tag>
               <el-input
              class="input-new-tag"
              v-if="scope.row.inputVisible"
              v-model="scope.row.inputValue"
              ref="saveTagInput"
              size="small"
              @keyup.enter.native="handleInputConfirm(scope.row)"
              @blur="handleInputConfirm(scope.row)">
            </el-input>
            <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
             </template>
           </el-table-column>
           <el-table-column type="index"></el-table-column>
           <el-table-column label="属性名称" prop="attr_name"></el-table-column>
           <el-table-column label="操作">
             <template slot-scope="scope">
               <el-button size="mini" type="primary" icon="el-icon-edit" @click="editShowDiaLog(scope.row.attr_id)">编辑</el-button>
               <el-button size="mini" type="danger" icon="el-icon-delete" @click="deleteParams(scope.row.attr_id)">删除</el-button>
             </template>
           </el-table-column>
           </el-table>
         </el-tab-pane>
         </el-tabs>
  </el-card>
   <!-- 添加参数对话框 -->
     <el-dialog
        :title="'添加' + titleText"
        :visible.sync="addDialogVisible"
        width="50%" @close="addDialogClosed">
        <!--添加参数表单-->
    <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
    <el-form-item :label="titleText" prop="attr_name">
    <el-input v-model="addForm.attr_name"></el-input>
  </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addParams">确 定</el-button>
  </span>
</el-dialog>
  <!-- 修改参数对话框 -->
    <el-dialog
        :title="'修改' + titleText"
        :visible.sync="editDialogVisible"
        width="50%" @close="editDialogClosed">
        <!--添加参数表单-->
    <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
    <el-form-item :label="titleText" prop="attr_name">
    <el-input v-model="editForm.attr_name"></el-input>
  </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editParams">确 定</el-button>
  </span>
</el-dialog>
    </div>
</template>
<script>
export default {
  data () {
    return {
      cateList: [],
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      selectedCateKeys: [],
      activeName: 'many',
      manyDataList: [],
      onlyDataList: [],
      addDialogVisible: false,
      addForm: {
        attr_name: ''
      },
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      editDialogVisible: false,
      editForm: {
        attr_name: ''
      },
      editFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品参数列表失败')
      }
      this.cateList = res.data
      this.$message.success('获取商品参数列表成功')
    },
    handleChange () {
      this.getParamsData()
    },
    handleTabClick () {
      this.getParamsData()
    },
    // 获取参数列表数据
    async getParamsData () {
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyDataList = []
        this.onlyDataList = []
      }
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
        params: { sel: this.activeName }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败')
      }
      console.log(res.data)
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        item.inputVisible = false
        item.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyDataList = res.data
      } else {
        this.onlyDataList = res.data
      }
    },
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    addParams () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) {
          return this.$message.error('添加商品参数失败')
        }
        this.$message.success('添加商品成功')
        this.getParamsData()
        this.addDialogVisible = false
      })
    },
    editDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    async editShowDiaLog (attrId) {
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${attrId}`, {
        params: { attr_sel: this.activeName }
      })
      if (res.meta.status !== 200) {
        this.$message.error('获取参数信息失败')
      }
      this.$message.success('获取参数信息成功')
      this.editForm = res.data
      this.editDialogVisible = true
    },
    editParams () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          { attr_name: this.editForm.attr_name, attr_sel: this.activeName }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('更新参数失败')
        }
        this.$message.success('更新参数成功')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    async deleteParams (attrId) {
      const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.error('已取消删除操作')
      }
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${attrId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败')
      }
      this.$message.success('删除参数成功')
      this.getParamsData()
    },
    handleInputConfirm (row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      this.saveParams(row)
    },
    showInput (row) {
      row.inputVisible = true
      // 当页面上的数据发生变化时就会调用下面的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    deletParams (i, row) {
      row.attr_vals.splice(i, 1)
      this.saveParams(row)
    },
    async saveParams (row) {
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(' ')
      })
      if (res.meta.status !== 200) {
        return this.$message.error('修改参数失败')
      }
      this.$message.success('修改参数成功')
    }
  },
  computed: {
    isBtnDisabled () {
      if (this.selectedCateKeys.length !== 3) {
        return true
      } else {
        return false
      }
    },
    // 当前选中的三级分类的Id
    cateId () {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null
    },
    titleText () {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态参数'
    }
  }
}
</script>
<style lang="less" scoped>
.el-row {
    margin: 15px 0;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 120px;
}
</style>
