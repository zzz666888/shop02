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
    <!-- 搜索区 -->
    <el-row :gutter="20">
    <el-col :span="8">
       <el-input placeholder="请输入内容">
       <el-button slot="append" icon="el-icon-search"></el-button>
       </el-input>
    </el-col>
    </el-row>
    <!-- 订单列表数据 -->
    <el-table :data="orderList" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="订单编号" prop='order_number'></el-table-column>
        <el-table-column label="订单价格" prop='order_price'></el-table-column>
        <el-table-column label="是否付款" prop='pay_status'>
            <template slot-scope="scope">
                <el-tag type="success" v-if="scope.row.pay_status=='1'">已付款</el-tag>
                <el-tag type="danger" v-else>未付款</el-tag>
            </template>
        </el-table-column>
        <el-table-column label="是否发货" prop='is_send'></el-table-column>
        <el-table-column label="下单时间" prop='create_time'>
            <template slot-scope="scope">
                {{scope.row.create_time | dateForm}}
            </template>
        </el-table-column>
        <el-table-column label="操作">
            <template>
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="showAddress"></el-button>
                <el-button size="mini" type="success" icon="el-icon-location" @click="progress"></el-button>
            </template>
        </el-table-column>
    </el-table>
    <!-- 分页区 -->
      <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page.sync="queryInfo.pagenum"
      :page-sizes="[1, 2, 5, 10]"
      :page-size="queryInfo.pagesize"
       layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
     </el-card>
     <!-- 添加地址对话框 -->
   <el-dialog
  title="修改地址"
  :visible.sync="addressVisible"
  width="50%">
  <el-form :model="addressForm" status-icon :rules="addressFormRules" ref="addressFormRef" label-width="100px">
  <el-form-item label="省市区/县" prop="address1">
    <el-cascader :options="cityData" v-model="addressForm.address1"></el-cascader>
  </el-form-item>
  <el-form-item label="详细地址" prop="address2">
    <el-input v-model="addressForm.address2"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="addressVisible = false">取 消</el-button>
    <el-button type="primary" @click="addressDialogClosed">确 定</el-button>
  </span>
   </el-dialog>
   <!-- 快递进度条 -->
     <el-dialog
  title="最新快递信息"
  :visible.sync="progressVisible"
  width="50%">
   <el-timeline>
    <el-timeline-item
      v-for="(activity, index) in progressInfo"
      :key="index"
      :timestamp="activity.time">
      {{activity.context}}
    </el-timeline-item>
  </el-timeline>
</el-dialog>
  </div>
</template>
<script>
import cityData from './citydata.js'
export default {
  data () {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 5
      },
      orderList: [],
      total: 0,
      addressVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [
          { required: true, message: '请选择省市区/县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请选择详细地址', trigger: 'blur' }
        ]
      },
      cityData,
      progressVisible: false,
      progressInfo: []
    }
  },
  created () {
    this.getOrderList()
  },
  methods: {
    async getOrderList () {
      const { data: res } = await this.$http.get('orders', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品订单失败')
      }
      this.$message.success('获取商品列表成功')
      this.total = res.data.total
      this.orderList = res.data.goods
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    showAddress () {
      this.addressVisible = true
    },
    addressDialogClosed () {
      this.addressVisible = false
      this.$refs.addressFormRef.resetFields()
    },
    async progress () {
      const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      if (res.meta.status !== 200) {
        return this.$message.error('获取快递信息失败')
      }
      this.$message.success('获取快递信息成功')
      this.progressInfo = res.data
      this.progressVisible = true
    }
  }
}
</script>
<style lang="less" scoped>
.el-cascader {
    width: 100%;
}
</style>
