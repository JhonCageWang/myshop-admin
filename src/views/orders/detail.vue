<template>
  <div >
    <main-panel title="订单详情" v-loading="loading" >
      <el-card v-if="order.user">
        <div slot="header">
          <span><i class="el-icon-fa-user">&nbsp;&nbsp;</i>客户信息</span>
        </div>
        <el-form label-position="left" label-width="90px">
          <el-form-item label="头像" class="iteminfo">
            <img :src="user.avatar" width="70" height="70" />
          </el-form-item>
          <el-form-item label="客户" class="iteminfo">
            <span>{{ user.nickname }}</span>
          </el-form-item>
          <el-form-item label="注册日期" class="iteminfo">
            <span>{{ user.registerTime }}</span>
          </el-form-item>
        </el-form>
      </el-card>
      <el-card style="margin-top: 10px">
        <div slot="header">
          <span><i class="el-icon-fa-info">&nbsp;&nbsp;</i>订单信息({{ order.id }})</span>
        </div>
        <el-table :data="orders" stripe border>
          <el-table-column prop="orderSn" label="订单号" min-width="150" show-overflow-tooltip></el-table-column>
          <el-table-column prop="payment" label="支付方式" width="80" show-overflow-tooltip>
            <template slot-scope="scope">
              微信支付
            </template>
          </el-table-column>
          <el-table-column prop="orderPrice" label="订单金额" width="80" align="right" show-overflow-tooltip></el-table-column>
          <el-table-column prop="actualPrice" label="实付款" width="80" align="right" show-overflow-tooltip></el-table-column>
          <el-table-column prop="orderExpress.shipperName" v-if="order.orderStatus >= 300" label="物流" width="120" show-overflow-tooltip></el-table-column>
          <el-table-column prop="orderExpress.logisticCode" v-if="order.orderStatus >= 300" label="运单号" width="100" show-overflow-tooltip></el-table-column>
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
          <el-table-column label="操作" width="240" v-if="order.orderStatus === 201  || order.orderStatus === 300">
            <template slot-scope="scope">
              <el-button type="primary" size="mini" icon="el-icon-fa-cube" @click="dialogDelivery = true" v-if="scope.row.orderStatus === 201  || scope.row.orderStatus === 300"  >发货</el-button>
              <el-button type="danger" size="mini" icon="el-icon-delete" @click="refound(scope.row) " v-if="scope.row.orderStatus === 201">退款</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-card>
      <el-card style="margin-top: 10px">
        <div slot="header">
          <span><i class="el-icon-fa-shopping-bag">&nbsp;&nbsp;</i>商品清单</span>
        </div>
        <el-table :data="goods" stripe border>
          <el-table-column label="图片" width="120" show-overflow-tooltip>
            <template slot-scope="scope">
              <img width="100" height="100" :src="scope.row.listPicUrl"/>
            </template>
          </el-table-column>
          <el-table-column prop="goodsName" label="商品名称" min-width="150" show-overflow-tooltip></el-table-column>
          <el-table-column prop="goodsSpecificationNameValue" label="规格" min-width="150" show-overflow-tooltip></el-table-column>
          <el-table-column prop="goodsSn" label="SKU" width="100" show-overflow-tooltip></el-table-column>
          <el-table-column prop="retailPrice" label="商品价格" width="100" align="right" show-overflow-tooltip></el-table-column>
          <el-table-column prop="number" label="数量" width="80" align="right" show-overflow-tooltip></el-table-column>
        </el-table>
      </el-card>
      <el-card style="margin: 10px 0">
        <div slot="header">
          <span><i class="el-icon-fa-truck">&nbsp;&nbsp;</i>收件人</span>
        </div>
        <el-table :data="addresses" stripe border>
          <el-table-column prop="name" label="收件人" width="80" show-overflow-tooltip></el-table-column>
          <el-table-column prop="mobile" label="联系方式" width="120" align="right" show-overflow-tooltip></el-table-column>
          <el-table-column prop="provinceName" label="省" width="80" show-overflow-tooltip></el-table-column>
          <el-table-column prop="cityName" label="市" width="80" show-overflow-tooltip></el-table-column>
          <el-table-column prop="districtName" label="地址" min-width="150" show-overflow-tooltip></el-table-column>
          <el-table-column prop="address" label="地址" min-width="150" show-overflow-tooltip></el-table-column>
        </el-table>
      </el-card>
      <!-- <el-card style="margin: 10px 0">
        <div slot="header">
          <span><i class="el-icon-fa-window-maximize">&nbsp;&nbsp;</i>发票信息</span>
        </div>
        <el-table :data="invoices" stripe border>
        <el-table-column prop="type" label="抬头类型" width="80" :formatter="formatter" show-overflow-tooltip></el-table-column>
        <el-table-column prop="name" label="抬头名称" width="100" show-overflow-tooltip></el-table-column>
        <el-table-column prop="tax_number" label="税号(纳税人识别号)" min-width="170" show-overflow-tooltip></el-table-column>
        </el-table>
      </el-card> -->
      <el-row>
        <router-link :to="{ path: '/orders' }">
          <el-button icon="el-icon-back" size="small" style="float: right">返回</el-button>
        </router-link>
      </el-row>
      <el-dialog
        v-if="!isMobile"
        style="width: 120%;"
        title="发货"
        :visible.sync="dialogDelivery"
        >
        <!-- <el-form ref="formDelivery" :model="formDelivery" label-width="80px" size="small">
          <el-form-item label="快递" prop="courierId" :rules="{required: true, message: '请选择发货快递'}">
            <el-select v-model="formDelivery.courierId" placeholder="请选择">
              <el-option v-for="item in couriers" :key="item.id" :label="item.name" :value="item.id"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="单号" prop="courierNo" :rules="{required: true, message: '请输入快递单号'}">
            <el-input v-model="formDelivery.courierNo"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button size="small" @click="dialogDelivery = false">取 消</el-button>
          <el-button size="small" type="primary" @click="delivery" :loading="loading">提 交</el-button>
        </span> -->
        <el-table :data="formDelivery" stripe border responsive="true">
          <el-table-column  label="快递公司" width="300" show-overflow-tooltip>
          <template slot-scope="scope">
            <el-select v-model="scope.row.courierId" placeholder="请选择">
              <el-option v-for="item in couriers" :key="item.id" :label="item.name" :value="item.id"></el-option>
            </el-select>
          </template>
          </el-table-column>
          <el-table-column prop="courierNo" label="快递单号" width="300" align="right" show-overflow-tooltip>
            <template slot-scope="scope">
              <el-input v-model="scope.row.courierNo"></el-input>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="240" >
            <template slot-scope="scope">
              <el-button type="primary" size="mini" icon="el-icon-fa-cube" @click="delivery(scope.row)" >保存</el-button>
              <el-button type="primary" size="mini" icon="el-icon-fa-cube" @click="addRow" >新增</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-dialog>

      <el-dialog
        v-if="isMobile"
        title="发货"
        :visible.sync="dialogDelivery"
        >
        <el-table :data="formDelivery" stripe border responsive="true">
          <el-table-column  label="快递公司"  >
          <template slot-scope="scope">
            <el-select v-model="scope.row.courierId" placeholder="请选择" style="display: block;block;margin-top:5px">
              <el-option v-for="item in couriers" :key="item.id" :label="item.name" :value="item.id"></el-option>
            </el-select>
            <el-input  v-model="scope.row.courierNo" placeHolder="快递单号" style="display: block;margin-top:5px"></el-input>
            <div style="margin-top:10px">
              <el-button type="primary" size="mini" icon="el-icon-fa-cube" @click="delivery(scope.row)" >保存</el-button>
              <el-button type="primary" size="mini" icon="el-icon-fa-cube" @click="addRow" >新增</el-button>
            </div>
          </template>
          </el-table-column>
        </el-table>
      </el-dialog>
    </main-panel>
  </div>
  
</template>
<style>
@media (max-width: 768px) {
  .el-dialog {
    width: 98%; /* 移动端宽度 */
    max-width: 100%; /* 最大宽度 */
    margin: 0; /* 去掉边距 */
  }
   
}

</style>
<script>
import * as api from 'api'
// import * as utils from 'utils'

export default {
  data () {
    return {
      isMobile:false,
      id: null,
      order: {},
      user: {},
      orders: [],
      goods: [],
      addresses: [],
      orderExpress: {},
      invoices: [],
      dialogDelivery: false,
      loading: false,
      couriers: [],
      formDelivery: [{
        courierId:1,
        courierNo:'',
        id:null
      }]
    }
  },
  mounted () {
    this.isMobile = window.innerWidth <= 768; 
    this.id = this.$route.params.id
    this.getOrdersInfo()
    this.getCouriersList()
  },
  
  methods: {
    touchStart(event) {
      debugger
      this.startX = event.touches[0].clientX;
    },
    touchEnd(event) {
      debugger
      this.endX = event.changedTouches[0].clientX;
      this.handleSwipe();
    },
    handleSwipe() {
      debugger
      if (this.endX - this.startX > 50) {
        // 向左滑动，返回上一个页面
        this.$router.go(-1);
      }
    },
    getOrdersInfo () {
      this.$http.get(api.ORDER + '/info?id=' + this.id).then(data => {
        console.log(data)
        this.order = data.data
        this.orders = [data.data]
        this.goods = data.data.goodsList
        this.addresses = [data.data.address]
        this.user = data.data.user
        // this.formDelivery.courierId = data.data.orderExpress.shipperId
        // this.formDelivery.courierNo = data.data.orderExpress.logisticCode
        if (data.data.orderExpressList && data.data.orderExpressList.length > 0) {
          this.orderExpress = data.data.orderExpressList[0]
          this.formDelivery = data.data.orderExpressList.map(r =>  {return {courierId:r.shipperId,courierNo:r.logisticCode,id:r.id}})
        }
      })
    },
    addRow() {
      this.formDelivery.push({
        courierId:1,
        courierNo:'',
        id:null
      })
    },
    refound (row) {
      this.$confirm(`确定要退款吗?`, '提示', {type: 'warning'}).then(() => {
        this.$http.post(api.ORDER + '/refund', {id: row.id}).then(data => {
          this.$notify({title: '成功', message: '退款成功', type: 'success'})
          this.getOrdersInfo()
        })
      }).catch(err => {
        console.log(err)
      })
    },
    getCouriersList () {
      this.$http.get(api.COURIER + '/list?page=1&size=1000').then(data => {
        this.couriers = data.data.list
        console.log(this.couriers)
      })
    },
    delivery (e) {
      console.info('axiba',e)
      this.loading = true
      this.$http.post(api.ORDER + '/' + this.id + '/delivery', e).then(data => {
        this.$notify({title: '成功', message: '发货成功', type: 'success'})
        this.loading = false
        this.dialogDelivery = false
        this.getOrdersInfo()
      }).catch(() => {
        this.loading = false
      })
    },
    formatter (row, column) {
      let types = {company: '单位', personal: '个人'}
      return types[row.type]
    }
  }
}
</script>

<style scoped>
.iteminfo {
  margin-bottom: 0;
  label {
    color: #99a9bf;
  }
}
</style>
