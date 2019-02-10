<template>
  <div class="cancel-order">
    <query-wrapper @userQuery="queryList">
      <Input class="query-item" v-model="queryArgs.keyword" placeholder="用户姓名/商品名称"/>
      <DatePicker
        class="query-item"
        type="datetime" placeholder="取消开始时间"
        clearable @on-change="cancelStartChange"></DatePicker>
      <DatePicker
        class="query-item"
        type="datetime" placeholder="取消结束时间"
        clearable @on-change="cancelEndChange"></DatePicker>
      <Select class="query-item" v-model="queryArgs.cancelType" placeholder="请选择取消类别" clearable>
        <Option v-for="item in cancelTypeList" :value="item.id" :key="item.id">{{ item.name }}</Option>
      </Select>
    </query-wrapper>
    <btn-wrapper @btnClick="btnClick"></btn-wrapper>
    <Table :columns="tableColumns" :loading="tableLoading" :data="tableData"
           @on-selection-change="tableSelectChange"></Table>
    <Page style="margin-top: 20px;text-align: center;" :current="pageNo" :total="total" show-elevator
          @on-change='changePage'></Page>
    <Modal
      v-model="lookModal"
      :mask-closable="false"
      title="查看订单详情">
      <order-edit v-if="lookModal" :detail="currentDetail" :isCancel="true"></order-edit>
      <div slot="footer">
      </div>
    </Modal>

  </div>
</template>
<script>
  import queryWrapper from '@/components/query-wrapper'
  import btnWrapper from '@/components/btn-wrapper'
  import orderEdit from './components/order-edit'
  import {message, table, page} from '@/common/js/mixins'
  import {allOrder} from '@/api/request'

  export default {
    data() {
      return {
        currentDetail: null,
        lookModal: false,
        selectIds: [],
        queryArgs: {
          keyword: '',
          startEndTime: '',
          cancelEndTime: '',
          cancelType: ''
        },
        cancelTypeList: [
          {
            id: 1,
            name: '付款超时'
          },
          {
            id: 2,
            name: '未达集采数量'
          },
          {
            id: 3,
            name: '用户取消'
          },
          {
            id: 4,
            name: '强制关闭'
          }
        ],
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
            title: '取消类别',
            key: 'createTimeStr'
          },
          {
            title: '取消时间',
            key: 'updateTimeStr'
          },
          {
            title: '操作',
            render: (h, params) => {
              return h('div', [
                h('Button', {
                  props: {
                    type: 'primary',
                    size: 'small'
                  },
                  style: {
                    margin: '5px'
                  },
                  on: {
                    click: () => {
                      allOrder.orderDetail({
                        purchaseOrderId: params.row.purchaseOrderId
                      }).then(data => {
                        if (data !== 'isError') {
                          this.currentDetail = data
                          this.lookModal = true
                        }
                      })
                    }
                  }
                }, '查看')
              ])
            }
          }
        ]
      }
    },
    components: {
      btnWrapper,
      queryWrapper,
      orderEdit
    },
    mixins: [message, table, page],
    created() {
      this.getAllOrder()
    },
    methods: {
      getAllOrder() {
        this.openTableLoading()
        allOrder.getOrderList({
          pageNo: this.pageNo,
          pageSize: 10,
          orderState: 9,
          ...this.queryArgs
        }).then(data => {
          this.closeTableLoading()
          if (data !== 'isError') {
            this.tableData = data.list
            this.total = data.total
          }
        })
      },
      tableSelectChange(selection) {
        let ids = []
        selection.forEach(item => {
          ids.push(item.purchaseOrderId)
        })
        this.selectIds = ids
      },
      btnClick(handleName) {
        switch (handleName) {
          case '批量发货':
            if (this.selectIds.length === 0) {
              return this.warningInfo('请选择操作对象')
            }
            this.$Modal.confirm({
              content: '确定要发货吗？',
              loading: true,
              onOk: () => {
                allOrder.sendOrder({
                  purchaseOrderId: this.selectIds.toString()
                }).then(data => {
                  if (data !== 'isError') {
                    this.successInfo('发货成功')
                    this.getAllOrder()
                    this.selectIds = []
                  }
                  this.$Modal.remove()
                })
              }
            })
            break
          case '下载订单模板':
            break
        }
      },
      queryList() {
        this.pageNo = 1
        this.getAllOrder()
      },
      changePage(no) {
        this.pageNo = no
        this.getAllOrder()
      },
      cancelStartChange(time) {
        this.cancelStartTime = time
      },
      cancelEndChange(time) {
        this.cancelEndTime = time
      }
    }
  }
</script>
<style lang="less">
</style>
