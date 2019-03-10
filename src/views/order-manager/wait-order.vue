<template>
  <div class="wait-order">
    <query-wrapper @userQuery="queryList">
      <Input class="query-item" v-model="queryArgs.goodsName" placeholder="产品名称"/>
      <Input class="query-item" v-model="queryArgs.goodsCode" placeholder="产品编码"/>
      <Input class="query-item" v-model="queryArgs.supplier" placeholder="供应商"/>
      <Select class="query-item" v-model="queryArgs.status" placeholder="产品状态">
        <Option v-for="item in statusList" :value="item.value" :key="item.value">{{ item.name }}</Option>
      </Select>
    </query-wrapper>
    <div class="btn-wrapper">
      <Button v-if="hasBtn('新建产品')" @click="openAddModal">新建产品</Button>
    </div>
    <Table :columns="tableColumns" :loading="tableLoading" :data="tableData"
           @on-selection-change="tableSelectChange"></Table>
    <Page style="margin-top: 20px;text-align: center;" :current="pageNo" :total="total" show-elevator
          @on-change='changePage'></Page>
    <Modal
      v-model="lookModal"
      :mask-closable="false"
      title="查看订单详情">
      <order-edit v-if="lookModal" :detail="currentDetail"></order-edit>
      <div slot="footer">
      </div>
    </Modal>
    <Modal
      v-model="addModal.isShow"
      width="575"
      :mask-closable="false"
      title="新建产品">
      <produce-edit v-if="addModal.isShow" ref="addEdit"></produce-edit>
      <div slot="footer" style="text-align: center">
        <Button type="primary" size="large" :loading="addModal.loading" @click="addConfirm">
          确定
        </Button>
        <Button type="error" size="large" @click="closeAddModal">取消</Button>
      </div>
    </Modal>
  </div>
</template>
<script>
  import queryWrapper from '@/components/query-wrapper'
  import btnWrapper from '@/components/btn-wrapper'
  import produceEdit from './components/produce-edit'
  import {message, table, page, addModal, writeModal} from '@/common/js/mixins'
  import {waitOrder} from '@/api/request'

  export default {
    data() {
      return {
        currentDetail: null,
        lookModal: false,
        selectIds: [],
        queryArgs: {
          keyword: '',
          addStartTime: '',
          addEndTime: '',
          saleDepartmentId: ''
        },
        statusList: [
          {
            value: 1,
            name: '启用'
          },
          {
            value: 0,
            name: '停用'
          }
        ],
        tableColumns: [
          {
            title: '产品名称',
            key: 'goodsName'
          },
          {
            title: '产品编码',
            key: 'goodsCode'
          },
          {
            title: '供应商',
            key: 'supplier'
          },
          {
            title: '单位',
            key: 'standard'
          },
          {
            title: '商品图片',
            render: (h, params) => {
              return h('div', params.row.goodsCount * params.row.salePrice)
            }
          },
          {
            title: '状态',
            render: (h, params) => {
              return h('div', params.row.status === 1 ? '启用' : '停用')
            }
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
                }, '查看'),
                h('Button', {
                  props: {
                    type: 'warning',
                    size: 'small'
                  },
                  style: {
                    margin: '5px'
                  },
                  on: {
                    click: () => {

                    }
                  }
                }, '编辑'),
                h('Button', {
                  props: {
                    type: 'error',
                    size: 'small'
                  },
                  style: {
                    margin: '5px'
                  },
                  on: {
                    click: () => {

                    }
                  }
                }, '删除')
              ])
            }
          }
        ]
      }
    },
    components: {
      btnWrapper,
      queryWrapper,
      produceEdit
    },
    mixins: [message, table, page, addModal, writeModal],
    created() {
      this.getAllOrder()
    },
    methods: {
      getAllOrder() {
        this.openTableLoading()
        waitOrder.getGoodsList({
          pageNo: this.pageNo,
          pageSize: 10,
          ...this.queryArgs
        }).then(data => {
          this.closeTableLoading()
          if (data !== 'isError') {
            this.tableData = data.list
            this.total = data.total
          }
        })
      },
      hasBtn (name) {
        return this.handleList[this.$route.name] && (this.handleList[this.$route.name].indexOf(name) > -1)
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
            let canSend = true
            this.selectIds.forEach(item => {
              let order = this.tableData.find(data => {
                return data.purchaseOrderId = item
              })
              if (order.haveMinCount === 0 || order.canSend === 0) {
                canSend = false
              }
            })
            if (!canSend) {
              return this.warningInfo('存在不能发货订单')
            }
            this.$Modal.confirm({
              content: '确定要发货吗？',
              loading: true,
              onOk: () => {
                allOrder.sendOrder({
                  purchaseOrderIds: this.selectIds.toString()
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
      addConfirm () {
        let returnData = this.$refs.addEdit.returnData()
        if (returnData) {
          this.openAddLoading()
          waitOrder.addGoods({
            ...returnData
          }).then(data => {
            this.closeAddLoading()
            if (data !== 'isError') {
              this.successInfo('新建产品成功')
              this.getAllOrder()
              this.closeAddModal()
            }
          }).catch(err => {
            this.closeAddLoading()
          })
        }
      }
    }
  }
</script>
<style lang="less">
</style>
