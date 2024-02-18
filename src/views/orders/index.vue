<template>
  <crud-panel title="订单列表">
    <el-row slot="condition">
      <el-input placeholder="请输入订单编号" size="small" prefix-icon="el-icon-search" v-model="orderSn" style="width: 200px;margin-right: 10px;" @keyup.enter.native="getOrdersList(true)" clearable></el-input>
      <el-input placeholder="请输入收货人" size="small" prefix-icon="el-icon-search" v-model="consignee" style="width: 200px;margin-right: 10px;" @keyup.enter.native="getOrdersList(true)" clearable></el-input>
      <el-select v-model="orderStatus" placeholder="请选择" size="small" @change="getOrdersList(true)"><el-option  v-for="item in options" :key="item.value" :label="item.label" :value="item.value"  clearable></el-option></el-select>
      <el-button type="primary" size="small" icon="el-icon-search" @click="getOrdersList(true)">查询</el-button>
    </el-row>
    <el-table :data="orders" stripe border v-loading="tbloading">
      <el-table-column type="index" width="50" align="center"></el-table-column>
      <el-table-column prop="orderSn" label="订单号" min-width="150" show-overflow-tooltip></el-table-column>
      <el-table-column prop="user.nickname" label="客户" width="80" show-overflow-tooltip></el-table-column>
      <el-table-column prop="payment" label="支付方式" width="80" show-overflow-tooltip>
        <template slot-scope="scope">
          微信支付
        </template>
      </el-table-column>
      <el-table-column prop="orderPrice" label="订单金额" width="80" align="right" show-overflow-tooltip></el-table-column>
      <el-table-column prop="actualPrice" label="实付款" width="80" align="right" show-overflow-tooltip></el-table-column>
      <el-table-column prop="orderStatus" label="状态" align="center" width="80">
        <template slot-scope="scope">
          <el-tag size="small" v-if="scope.row.orderStatus === 0">未付款</el-tag>
          <el-tag type="warning" size="small" v-if="scope.row.orderStatus === 101">已取消</el-tag>
          <el-tag type="danger" size="small" v-if="scope.row.orderStatus === 102">已删除</el-tag>
          <el-tag type="success" size="small" v-if="scope.row.orderStatus === 201">已付款</el-tag>
          <el-tag type="info" size="small" v-if="scope.row.orderStatus === 300">已发货</el-tag>
          <el-tag type="info" size="small" v-if="scope.row.orderStatus === 301">待评价</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="addTime" label="创建日期" width="170" show-overflow-tooltip></el-table-column>
      <el-table-column label="操作" width="120">
        <template slot-scope="scope">
          <router-link :to="{ path: '/orders/detail/' + scope.row.id }">
            <el-button type="primary" size="mini" icon="el-icon-view">详情</el-button>
          </router-link>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      slot="page"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="page.page"
      :page-sizes="[10, 20, 30, 40, 50]"
      :page-size="page.size"
      layout="total, sizes, prev, pager, next, jumper"
      :total="page.total">
    </el-pagination>
  </crud-panel>
</template>

<script>
import * as api from 'api'

export default {
  data () {
    return {
      orderSn: '',
      tbloading: false,
      orders: [],
      consignee: '',
      orderStatus: '',
      page: {
        size: 10,
        page: 1,
        total: 0
      },
      options: [
        {
          value: '',
          label: '全部'
        },
        {
          value: '0',
          label: '待付款'
        }, {
          value: '201',
          label: '已付款'
        }, {
          value: '101',
          label: '已取消'
        }, {
          value: '300',
          label: '已发货'
        }
      ]
    }
  },
  mounted () {
    this.getOrdersList()
  },
  methods: {
    getOrdersList (search = false) {
      console.log('asdfasdf', this.orderStatus)
      this.tbloading = true
      if (search) {
        this.page.page = 1
      }
      this.$http.get(api.ORDER + `/list?page=${this.page.page}&size=${this.page.size}&orderSn=${this.orderSn}&consignee=${this.consignee}&orderStatus=${this.orderStatus}`).then(data => {
        this.orders = data.data.list
        this.page.total = data.data.total
        this.tbloading = false
      }).catch(err => {
        this.tbloading = false
        this.$message.error(err + '')
      })
    },
    handleSizeChange (val) {
      this.page.size = val
      this.getordersList()
    },
    handleCurrentChange (val) {
      this.page.page = val
      this.getOrdersList()
    }
  }
}
</script>

<style scoped>

</style>
