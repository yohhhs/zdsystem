<template>
  <div class="cancel-order">
    <query-wrapper @userQuery="queryList">
      <Input class="query-item" v-model="queryArgs.inviteCode" placeholder="邀请码" clearable/>
      <Input class="query-item" v-model="queryArgs.goodsName" placeholder="商品名称" clearable/>
      <Input class="query-item" v-model="queryArgs.goodsCode" placeholder="商品编码" clearable/>
      <DatePicker
        class="query-item"
        type="datetime" placeholder="开始时间"
        clearable :editable="false" @on-change="orderStartChange"></DatePicker>
      <DatePicker
        class="query-item"
        type="datetime" placeholder="结束时间"
        clearable :editable="false" @on-change="orderEndChange"></DatePicker>
      <!--<Select class="query-item" v-model="queryArgs.solicitState" placeholder="请选择状态" clearable>
        <Option v-for="item in orderStatusList" :value="item.id" :key="item.id">{{ item.name }}</Option>
      </Select>-->
    </query-wrapper>
    <div class="btn-wrapper">
      <Button v-if="hasBtn('导出')" @click="exportExcel">导出</Button>
    </div>
    <Table :columns="tableColumns" :loading="tableLoading" :data="tableData" @on-selection-change="tableSelectChange"></Table>
    <Page style="margin-top: 20px;text-align: center;" :current="pageNo" :total="total" show-elevator @on-change='changePage'></Page>
  </div>
</template>
<script>
  import queryWrapper from '@/components/query-wrapper'
  import btnWrapper from '@/components/btn-wrapper'
  import orderEdit from './components/order-edit'
  import { message, table, page, addModal, writeModal } from '@/common/js/mixins'
  import { getOrderList } from '@/api/request'
  import {baseUrl} from "../../common/js/config";

  export default {
    data () {
      return {
        listDetail: {
          col: [
            {
              title: '姓名',
              key: 'memberName'
            },
            {
              title: '电话',
              key: 'memberMobile'
            },
            {
              title: '数量',
              key: 'goodsCount'
            },
            {
              title: '区部',
              width: 300,
              key: 'addressDetail'
            }
          ],
          data: [],
          loading: false,
          count: 0
        },
        uploadModal: false,
        currentDetail: null,
        lookModal: false,
        selectIds: [],
        changeStatusModal: {
          isShow: false,
          loading: false
        },
        deleteModal: {
          isShow: false,
          loading: false
        },
        queryArgs: {
          inviteCode: '',
          goodsName: '',
          goodsCode: '',
          startTime: '',
          endTime: ''
        },
        orderStatusList: [
          {
            id: 1,
            name: '未开始'
          },
          {
            id: 2,
            name: '进行中'
          },
          {
            id: 3,
            name: '已完成'
          }
        ],
        tableColumns: [
          {
            title: '邀请码',
            width: 120,
            key: 'inviteCode'
          },
          {
            title: '产品编码',
            key: 'goodsCode'
          },
          {
            title: '产品名称',
            key: 'goodsName'
          },
          {
            title: '供应商',
            key: 'supplier'
          },
          {
            title: '征订开始时间',
            key: 'startTime'
          },
          {
            title: '征订结束时间',
            key: 'endTime'
          },
          {
            title: '状态',
            render: (h, params) => {
              let status = ['未开始', '进行中', '已完成']
              return h('div', params.row.orderState === 9 ? '已取消' : status[params.row.solicitState])
            }
          },
          {
            title: '数量',
            key: 'goodsCount'
          }
        ]
      }
    },
    components: {
      btnWrapper,
      queryWrapper,
      orderEdit
    },
    mixins: [message, table, page, addModal, writeModal],
    created () {
      this.getAllOrder()
    },
    methods: {
      hasBtn (name) {
        return this.handleList[this.$route.name] && (this.handleList[this.$route.name].indexOf(name) > -1)
      },
      getAllOrder () {
        this.openTableLoading()
        getOrderList({
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
      tableSelectChange (selection) {
        let ids = []
        selection.forEach(item => {
          ids.push(item.purchaseOrderId)
        })
        this.selectIds = ids
      },
      changeDetailPage (page) {
        this.listDetail.page = page
        this.getListDetail()
      },
      getListDetail () {
        this.listDetail.loading = true
        allOrder.getOrderList({
          pageNo: 1,
          pageSize: 100,
          solicitGoodsId: this.currentDetail.goodsId
        }).then(data => {
          this.listDetail.loading = false
          if (data !== 'isError') {
            this.listDetail.data = data.list
            this.listDetail.count = 0
            if (data.list.length > 0) {
              data.list.forEach(item => {
                this.listDetail.count += item.goodsCount
              })
            }
          }
        }).catch(err => {
          this.listDetail.loading = false
        })
      },
      // btnClick (handleName) {
      //   switch (handleName) {
      //     case '强制关闭':
      //       if (this.selectIds.length === 0) {
      //         return this.warningInfo('请选择操作对象')
      //       }
      //       this.$Modal.confirm({
      //         content: '确定要关闭吗？',
      //         loading: true,
      //         onOk: () => {
      //           allOrder.closeOrder({
      //             purchaseOrderId: this.selectIds.toString()
      //           }).then(data => {
      //             if (data !== 'isError') {
      //               this.successInfo('关闭成功')
      //               this.getAllOrder()
      //             }
      //               this.$Modal.remove()
      //           })
      //         }
      //       })
      //       break
      //     case '导出订单':
      //       window.location.href = 'https://www.topasst.com/cms/purchaseOrder/getPurchaseOrderExcelList?orderState=' + this.queryArgs.orderState
      //       break
      //     case '导入订单':
      //       this.uploadModal = true
      //       break
      //   }
      // },
      queryList () {
        this.pageNo = 1
        this.getAllOrder()
      },
      changePage (no) {
        this.pageNo = no
        this.getAllOrder()
      },
      orderStartChange (time) {
        this.queryArgs.startTime = time
      },
      orderEndChange (time) {
        this.queryArgs.endTime = time
      },
      uploadSuccess (res) {
        if (res.statusCode == 200) {
          this.successInfo('导入成功')
        } else if (res.statusCode == 412) {
          window.location.href = res.data
        }
        this.getAllOrder()
      },
      formatHandle () {
        this.warningInfo('文件格式不正确')
      },
      openChangeStatusModal () {
        this.changeStatusModal.isShow = true
      },
      openDeleteModal () {
        this.deleteModal.isShow = true
      },
      exportExcel () {
        let params = []
        for (let k in this.queryArgs) {
          if (this.queryArgs[k] === 0) {
            params.push(k + '=' + this.queryArgs[k])
          } else {
            params.push(k + '=' + (this.queryArgs[k] ? this.queryArgs[k] : ''))
          }
        }
        window.location.href = baseUrl + '/purchaseOrder/getPurchaseOrderExcelList?' + params.join('&')
      }
    }
  }
</script>
<style lang="less" scoped>
  .btn-wrapper {
    margin-bottom: 5px;
    padding: 8px 30px;
    background-color: #fff;
  }
</style>
