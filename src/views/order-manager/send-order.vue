<template>
  <div class="send-order">
    <div class="query-wrapper">
      <Select class="query-item" v-model="saleId" placeholder="请选择营业部" @on-change="changeSale">
        <Option v-for="item in saleList" :value="item.infoId" :key="item.infoId">{{ item.infoValue }}</Option>
      </Select>
      <Select class="query-item" v-model="collectGoodsId" placeholder="请选择集采商品" @on-change="changeGoods">
        <Option v-for="item in collectGoodsList" :value="item.infoId" :key="item.infoId">{{ item.infoValue }}</Option>
      </Select>
    </div>
    <btn-wrapper @btnClick="btnClick"></btn-wrapper>
    <Table :columns="tableColumns" :loading="tableLoading" :data="tableData"></Table>
    <Page style="margin-top: 20px;text-align: center;" :current="pageNo" :total="total" show-elevator @on-change='changePage'></Page>
  </div>
</template>

<script>
  import { message, table, page } from '@/common/js/mixins'
  import btnWrapper from '@/components/btn-wrapper'
  import {sendOrderPage} from '@/api/request'

  export default {
    name: "send-order",
    data () {
      return {
        saleId: '',
        collectGoodsId: '',
        saleList: [],
        collectGoodsList: [],
        tableColumns: [
          {
            title: '订单编号',
            key: 'purchaseOrderNumber'
          },
          {
            title: '集采商品编号',
            key: 'purchaseGoodsNumber'
          },
          {
            title: '用户',
            key: 'agentMemberName'
          },
          {
            title: '营业部',
            key: 'saleDepartmentName'
          },
          {
            title: '订单金额',
            render: (h, params) => {
              return h('div', params.row.goodsCount * params.row.salePrice)
            }
          },
          {
            title: '商品名称',
            key: 'goodsName'
          },
          {
            title: '数量',
            key: 'goodsCount'
          },
          {
            title: '支付方式',
            render: (h, params) => {
              return h('div', params.row.payType === 1 ? '线上' : '线下')
            }
          },
          {
            title: '起订量',
            render: (h, params) => {
              return h('div', params.row.haveMinCount === 1 ? '达标' : '未达标')
            }
          },
          {
            title: '下单时间',
            key: 'createTimeStr'
          },
          {
            title: '集采结束时间',
            key: 'purchaseEndTime'
          }
        ]
      }
    },
    components: {
      btnWrapper
    },
    mixins: [message, table, page],
    created () {
      this.getSaleList()
    },
    methods: {
      getSaleList () {
        sendOrderPage.getNotSendSale().then(data => {
          if (data !== 'isError') {
            this.saleList = data
          }
        })
      },
      getGoodsList () {
        sendOrderPage.getNotSendOrder({
          saleDepartmentId: this.saleId
        }).then(data => {
          if (data !== 'isError') {
            this.collectGoodsList = data
          }
        })
      },
      getOrderList () {
        sendOrderPage.getOrderList({
          pageNo: this.pageNo,
          pageSize: 10,
          purchaseGoodsId: this.collectGoodsId
        }).then(data => {
          if (data !== 'isError') {
            this.tableData = data.list
            this.total = data.total
          }
        })
      },
      changeSale () {
        this.getGoodsList()
      },
      changeGoods () {
        this.getOrderList()
      },
      btnClick (handleName) {
        switch (handleName) {
          case '一键发货':
              sendOrderPage.oneTapSend({
                purchaseGoodsId: this.collectGoodsId
              }).then(data => {
                if (data !== 'isError') {
                  this.successInfo('发货成功')
                  this.getOrderList()
                }
              })
            break
        }
      },
      changePage (no) {
        this.pageNo = no
        this.getOrderList()
      }
    }
  }
</script>

<style lang="less" scoped>
  .query-wrapper {
    margin-bottom: 10px;
    padding: 10px;
    background-color: #fff;
  }
</style>
