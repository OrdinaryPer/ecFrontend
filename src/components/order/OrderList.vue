<template>
  <div>
    <main-nav>
      <template v-slot:otherBreadcrumb>
        <nav-item>订单管理</nav-item>
        <nav-item>订单列表</nav-item>
      </template>
    </main-nav>
    <el-card shadow="always" :body-style="{ padding: '20px' }">
      <el-row :gutter="20">
        <el-col :span="8" :offset="0">
          <el-input placeholder="请输入内容">
            <el-button icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <el-table :data="orderList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="订单编号" prop="order_number"></el-table-column>
        <el-table-column label="订单价格" prop="order_price"></el-table-column>
        <el-table-column label="是否付款" prop="pay_status">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status == '1'">已付款</el-tag>
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send"></el-table-column>
        <el-table-column label="下单时间">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="">
            <el-button type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showModifyBox"></el-button>
            <el-button type="success"
              icon="el-icon-location"
              size="mini"
              @click="showProgressBox"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 20, 30]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
    </el-pagination>
    </el-card>
    <!-- 修改对话框 -->
    <el-dialog
      title="修改地址"
      :visible.sync="addressDialogVisible"
      width="50%"
      @close="addressDialogCloesed">
      <el-form :model="addressForm"
        :rules="addressFormRules"
        ref="addressFormRef"
        label-width="100px">
        <el-form-item label="省市区/县" prop="address1">
          <el-cascader :options="citydata"
            v-model="addressForm.address1"
            popper-class="cascader-panel">
          </el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
      <el-button @click="dialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 物流信息对话框 -->
    <el-dialog
      title="物流进度"
      :visible.sync="progressDialogVisible"
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
import citydata from '../../assets/js/citydata.js'
import MainNav from '@/components/navigation/MainNav'
import NavItem from '@/components/navigation/NavItem'

export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      orderList: [],
      addressDialogVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [
          {
            required: true,
            message: '请选择省市区县',
            trigger: 'blur'
          }
        ],
        address2: [
          {
            required: true,
            message: '请填写详细地址',
            trigger: 'blur'
          }
        ]
      },
      citydata,
      progressDialogVisible: false,
      progressInfo: []
    }
  },

  created () {
    this.getOrderList()
  },

  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单列表失败')
      }
      this.total = res.data.total
      this.orderList = res.data.goods
    },

    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },

    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },

    showModifyBox() {
      this.addressDialogVisible = true
    },

    addressDialogCloesed() {
      this.$refs.addressFormRef.resetFields()
    },

    async showProgressBox() {
      const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.message)
      }
      console.log(res.data)
      this.progressInfo = res.data
      this.addressDialogVisible = true
    }
  },

  components: {
    MainNav,
    NavItem
  }
}
</script>

<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
