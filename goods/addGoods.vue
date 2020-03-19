<template>
    <div>
    <!-- 面包屑导航区 -->
     <el-breadcrumb separator-class="el-icon-arrow-right">
     <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
     <el-breadcrumb-item>商品管理</el-breadcrumb-item>
     <el-breadcrumb-item>添加商品</el-breadcrumb-item>
     </el-breadcrumb>
     <!-- 卡片区域-->
     <el-card>
        <el-alert title="警告提示的文案" type="info" show-icon  center></el-alert>
        <!--步骤条区  -->
        <el-steps :space="200" :active="activeIndex - 0" finish-status="success" simple style="margin-top: 20px" align-center>
        <el-step title="基本信息 1" icon="el-icon-upload" ></el-step>
        <el-step title="商品参数 2" icon="el-icon-upload">
        </el-step>
        <el-step title="商品属性 3" icon="el-icon-upload"></el-step>
        <el-step title="商品图片 4" icon="el-icon-picture"></el-step>
        <el-step title="商品内容 5" icon="el-icon-upload"></el-step>
        <el-step title="完成 6" icon="el-icon-upload"></el-step>
        </el-steps>
        <!-- table栏区域 -->
    <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
    <el-tabs v-model="activeIndex" :tab-position="'left'" :before-leave="defaultLeave" @tab-click="tabClicked">
    <el-tab-pane label="基本信息" name="0">
        <el-form-item label="商品名称" prop="goods_name">
        <el-input v-model="addForm.goods_name"></el-input>
        </el-form-item>
        <el-form-item label="商品价格" prop="goods_price">
        <el-input v-model="addForm.goods_price" type="number"></el-input>
        </el-form-item>
        <el-form-item label="商品数量" prop="goods_number" >
        <el-input v-model="addForm.goods_number" type="number"></el-input>
        </el-form-item>
        <el-form-item label="商品重量" prop="goods_weight" >
        <el-input v-model="addForm.goods_weight" type="number"></el-input>
        </el-form-item>
          <!-- 级联选择区 -->
          <el-form-item label="请选择商品分类"></el-form-item>
           <el-cascader
            expand-trigger="hover"
            v-model="addForm.goods_cat"
            :options="cateList"
            :props="cateProps"
            @change="handleChange">
            </el-cascader>
    </el-tab-pane>
    <el-tab-pane label="商品参数" name="1">
         <el-form-item :label="item.attr_name" v-for="item in manyTabData" :key="item.attr_id">
           <!-- 复选框组 -->
           <el-checkbox-group v-model="item.attr_vals">
           <el-checkbox :label="cb" v-for="(cb, i) in  item.attr_vals " :key="i" border></el-checkbox>
           </el-checkbox-group>
         </el-form-item>
    </el-tab-pane>
    <el-tab-pane label="商品属性" name="2">
         <el-form-item :label="item.attr_name" v-for="item in onlyTabData" :key="item.attr_id">
           <el-input v-model="item.attr_vals"></el-input>
         </el-form-item>
    </el-tab-pane>
    <el-tab-pane label="商品图片" name="3">
        <el-upload
           :action="uploadURL"
           :on-preview="handlePreview"
           :on-remove="handleRemove"
           list-type="picture" :headers="headersObj" :on-success="uploadSuccess">
           <el-button size="small" type="primary">点击上传</el-button>
        </el-upload>
    </el-tab-pane>
    <el-tab-pane label="商品内容" name="4">
      <!-- 富文本编辑器组件 -->
      <quill-editor v-model="addForm.goods_introduce"></quill-editor>
      <!-- 添加商品的按钮 -->
      <el-button type="success" class="add" @click="addGoods">添加商品</el-button>
    </el-tab-pane>
    </el-tabs>
    </el-form>
     </el-card>
      <!-- 图片预览对话框 -->
     <el-dialog
       title="图片预览" :visible.sync="previewVisible" width="50%" class="previewPic">
       <img :src="previewPath" alt="">
      </el-dialog>
    </div>
</template>
<script>
import _ from 'lodash'
export default {
  data () {
    return {
      activeIndex: '0',
      addForm: {
        goods_name: '',
        goods_price: '',
        goods_number: '',
        goods_weight: '',
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ]
      },
      cateList: [],
      cateProps: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      manyTabData: [],
      onlyTabData: [],
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      headersObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      previewVisible: false,
      previewPath: ''
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        this.$.message.error('获取商品列表失败')
      }
      this.cateList = res.data
    },
    handleChange () {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    defaultLeave (activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请选选择商品分类')
        return false
      }
    },
    async tabClicked () {
      if (this.activeIndex === '1') {
        if (this.manyTabData.length === 0) {
          const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
            params: { sel: 'many' }
          })
          if (res.meta.status !== 200) {
            return this.$message.error('获取商品动态参数失败')
          }
          console.log(res.data)
          res.data.forEach(item => {
            item.attr_vals = item.attr_vals.length === 0 ? []
              : item.attr_vals.split(' ')
          })
          this.$message.success('获取商品动态参数成功')
          this.manyTabData = res.data
        }
      } else if (this.activeIndex === '2') {
        if (this.onlyTabData.length === 0) {
          const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
            params: { sel: 'only' }
          })
          if (res.meta.status !== 200) {
            return this.$message.error('获取商品属性失败')
          }
          this.$message.success('获取商品属性成功')
          this.onlyTabData = res.data
        }
      }
    },
    handlePreview (file) {
      console.log(file)
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    handleRemove (file) {
      const filePath = file.response.data.tmp_path
      const i = this.addForm.pics.findIndex(item => item.pics === filePath)
      this.addForm.pics.splice(i, 1)
    },
    uploadSuccess (reseponse) {
      const picInfo = { pic: reseponse.data.tmp_path }
      this.addForm.pics.push(picInfo)
    },
    addGoods () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('请输入必填的验证项')
        }
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        // 处理动态参数
        this.manyTabData.forEach(item => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals.join(' ') }
          this.addForm.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTabData.forEach(item => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals }
          this.addForm.attrs.push(newInfo)
        })
        form.attrs = this.addForm.attrs
        console.log(form)
        // 发送请求添加商品
        const { data: res } = await this.$http.post('goods', form)
        if (res.meta.status !== 201) {
          return this.$message.error('添加商品失败')
        }
        this.$message.success('添加商品成功')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    cateId () {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>
<style lang="less" scoped>
.el-checkbox {
  margin-right: 10px;
}
.previewPic {
  width: 100%;
}
.add {
  margin-top: 15px;
}
</style>
