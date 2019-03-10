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
    <Modal
      v-model="writeModal.isShow"
      width="575"
      :mask-closable="false"
      title="修改产品">
      <produce-edit v-if="writeModal.isShow" ref="writeEdit" isWrite :detail="currentDetail"></produce-edit>
      <div slot="footer" style="text-align: center">
        <Button type="primary" size="large" :loading="writeModal.loading" @click="writeConfirm">
          确定
        </Button>
        <Button type="error" size="large" @click="closeWriteModal">取消</Button>
      </div>
    </Modal>
    <Modal
      v-model="lookImgModal.isShow"
      width="575"
      footer-hide
      title="查看图片">
      <div class="modal-img">
        <img :src="currentLookImg" alt="">
      </div>
      <div slot="footer" style="text-align: center">
      </div>
    </Modal>
    <Modal
      v-model="deleteModal.isShow"
      :mask-closable="false"
      title="删除产品">
      <p style="text-align: center;font-size: 14px;line-height: 60px">确认删除当前产品吗</p>
      <div slot="footer" style="text-align: center">
        <Button type="error" :loading="deleteModal.loading" @click="deleteConfirm">确认删除</Button>
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
        currentLookImg: '',
        lookImgModal: {
          isShow: false
        },
        deleteModal: {
          isShow: false,
          loading: false
        },
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
              return h('div', {
                class: 'table-img',
                on: {
                  click: () => {
                    if (params.row.goodsImage) {
                      this.currentLookImg = params.row.goodsImage
                      this.lookImgModal.isShow = true
                    }
                  }
                }
              }, [h('img', {
                attrs: {
                  src: params.row.goodsImage
                }
              })])
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
                    type: 'warning',
                    size: 'small'
                  },
                  style: {
                    margin: '5px'
                  },
                  on: {
                    click: () => {
                      waitOrder.updateGoodsStatus({
                        goodsId: params.row.goodsId,
                        status: params.row.status === 1 ? 0 : 1
                      }).then(data => {
                        if (data !== 'isError') {
                          this.successInfo('更改状态成功')
                          this.getAllOrder()
                        }
                      })
                    }
                  }
                }, params.row.status === 1 ? '停用' : '启用'),
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
                      this.currentDetail = params.row
                      this.openWriteModal()
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
                      this.currentDetail = params.row
                      this.deleteModal.isShow = true
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
      },
      writeConfirm () {
        let returnData = this.$refs.writeEdit.returnData()
        if (returnData) {
          delete returnData.goodsCode
          this.openWriteLoading()
          waitOrder.updateGoods({
            goodsId: this.currentDetail.goodsId,
            ...returnData
          }).then(data => {
            this.closeWriteLoading()
            if (data !== 'isError') {
              this.successInfo('修改产品成功')
              this.getAllOrder()
              this.closeWriteModal()
            }
          }).catch(err => {
            this.closeWriteLoading()
          })
        }
      },
      deleteConfirm () {
        this.deleteModal.loading = true
        waitOrder.deleteGoods({
          goodsId: this.currentDetail.goodsId
        }).then(data => {
          if (data !== 'isError') {
            this.successInfo('删除产品成功')
            this.correctPageNo()
            this.getAllOrder()
            this.deleteModal.isShow = false
          }
          this.deleteModal.loading = false
        }).catch(err => {
          this.deleteModal.loading = false
        })
      }
    }
  }
</script>
<style lang="less">
  .table-img {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 80px;
    height: 80px;
    cursor: pointer;
    img {
      max-width: 100%;
      man-height: 100%;
    }
  }
  .modal-img {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 543px;
    height: 543px;
    background-color: #eee;
    img {
      max-width: 100%;
      man-height: 100%;
    }
  }
</style>
