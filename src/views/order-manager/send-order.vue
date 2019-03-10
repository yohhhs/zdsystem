<template>
  <div class="send-order">
    <query-wrapper @userQuery="queryList">
      <Input class="query-item" v-model="queryArgs.inviteCode" placeholder="邀请码"/>
      <Input class="query-item" v-model="queryArgs.province" placeholder="省"/>
      <Input class="query-item" v-model="queryArgs.city" placeholder="市"/>
    </query-wrapper>
    <div class="btn-wrapper">
      <Button v-if="hasBtn('添加征订')" @click="openAddModal">添加征订</Button>
    </div>
    <Table :columns="tableColumns" :loading="tableLoading" :data="tableData"></Table>
    <Page style="margin-top: 20px;text-align: center;" :current="pageNo" :total="total" show-elevator @on-change='changePage'></Page>
    <Modal
      v-model="addModal.isShow"
      width="575"
      :mask-closable="false"
      title="新建征订">
      <solicit-edit v-if="addModal.isShow" ref="addEdit" :codeList="codeList" :goodsList="goodsList"></solicit-edit>
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
      title="修改征订">
      <solicit-edit v-if="writeModal.isShow" ref="writeEdit" isWrite :detail="currentDetail" :codeList="codeList" :goodsList="goodsList"></solicit-edit>
      <div slot="footer" style="text-align: center">
        <Button type="primary" size="large" :loading="writeModal.loading" @click="writeConfirm">
          确定
        </Button>
        <Button type="error" size="large" @click="closeWriteModal">取消</Button>
      </div>
    </Modal>
    <Modal
      v-model="deleteModal.isShow"
      :mask-closable="false"
      title="删除征订">
      <p style="text-align: center;font-size: 14px;line-height: 60px">确认删除当前征订吗</p>
      <div slot="footer" style="text-align: center">
        <Button type="error" :loading="deleteModal.loading" @click="deleteConfirm">确认删除</Button>
      </div>
    </Modal>
  </div>
</template>

<script>
  import queryWrapper from '@/components/query-wrapper'
  import { message, table, page, addModal, writeModal } from '@/common/js/mixins'
  import btnWrapper from '@/components/btn-wrapper'
  import {sendOrderPage} from '@/api/request'
  import solicitEdit from './components/solicit-edit'

  export default {
    name: "send-order",
    data () {
      return {
        currentDetail: null,
        saleId: '',
        collectGoodsId: '',
        codeList: [],
        goodsList: [],
        collectGoodsList: [],
        queryArgs: {
          inviteCode: '',
          province: '',
          city: ''
        },
        deleteModal: {
          isShow: false,
          loading: false
        },
        tableColumns: [
          {
            title: '邀请码',
            key: 'inviteCode'
          },
          {
            title: '产品名称',
            key: 'goodsName'
          },
          {
            title: '开始时间',
            key: 'startTime'
          },
          {
            title: '省',
            key: 'province'
          },
          {
            title: '市',
            key: 'city'
          },
          {
            title: '结束时间',
            key: 'endTime'
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
                      sendOrderPage.updateSolicitStatus({
                        solicitGoodsId: params.row.solicitGoodsId,
                        status: params.row.status === 1 ? 0 : 1
                      }).then(data => {
                        if (data !== 'isError') {
                          this.successInfo('更改状态成功')
                          this.getOrderList()
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
      solicitEdit
    },
    mixins: [message, table, page, addModal, writeModal],
    created () {
      this.getCodeList()
      this.getGoodsList()
      this.getOrderList()
    },
    methods: {
      hasBtn (name) {
        return this.handleList[this.$route.name] && (this.handleList[this.$route.name].indexOf(name) > -1)
      },
      getCodeList () {
        sendOrderPage.getInviteCodeList({
          pageSize: 1000000,
          pageNo: 1
        }).then(data => {
          if (data !== 'isError') {
            this.codeList = data.list
          }
        })
      },
      getGoodsList () {
        sendOrderPage.getGoodsList({
          pageSize: 1000000,
          pageNo: 1
        }).then(data => {
          if (data !== 'isError') {
            this.goodsList = data.list
          }
        })
      },
      getOrderList () {
        sendOrderPage.getSolicitGoodsList({
          pageNo: this.pageNo,
          pageSize: 10,
          ...this.queryArgs
        }).then(data => {
          if (data !== 'isError') {
            this.tableData = data.list
            this.total = data.total
          }
        })
      },
      addConfirm () {
        let returnData = this.$refs.addEdit.returnData()
        if (returnData) {
          this.openAddLoading()
          sendOrderPage.addSolicitGoods({
            ...returnData
          }).then(data => {
            this.closeAddLoading()
            if (data !== 'isError') {
              this.successInfo('新建产品成功')
              this.getOrderList()
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
          delete returnData.inviteCodeId
          delete returnData.goodsId
          this.openWriteLoading()
          sendOrderPage.updateSolicitGoods({
            solicitGoodsId: this.currentDetail.solicitGoodsId,
            ...returnData
          }).then(data => {
            this.closeWriteLoading()
            if (data !== 'isError') {
              this.successInfo('修改产品成功')
              this.getOrderList()
              this.closeWriteModal()
            }
          }).catch(err => {
            this.closeWriteLoading()
          })
        }
      },
      deleteConfirm () {
        this.deleteModal.loading = true
        sendOrderPage.deleteSolicitGoods({
          solicitGoodsId: this.currentDetail.solicitGoodsId
        }).then(data => {
          if (data !== 'isError') {
            this.successInfo('删除产品成功')
            this.correctPageNo()
            this.getOrderList()
            this.deleteModal.isShow = false
          }
          this.deleteModal.loading = false
        }).catch(err => {
          this.deleteModal.loading = false
        })
      },
      changePage (no) {
        this.pageNo = no
        this.getOrderList()
      },
      queryList () {
        this.pageNo = 1
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
