<template>
  <div class="solicit-edit">
    <div class="modal-input-item">
      <p class="label">邀请码：</p>
      <div style="width: 350px">
        <Select v-model="formData.inviteCodeId" :disabled="isWrite" placeholder="请选择邀请码" filterable>
          <Option v-for="item in codeList" :value="item.inviteCodeId" :key="item.inviteCodeId">{{ item.inviteCode }}</Option>
        </Select>
      </div>
    </div>
    <div class="modal-input-item">
      <p class="label">产品：</p>
      <div style="width: 350px">
        <Select v-model="formData.goodsId" :disabled="isWrite" placeholder="请选择产品">
          <Option v-for="item in goodsList" :value="item.goodsId" :key="item.goodsId">{{ item.goodsName }}</Option>
        </Select>
      </div>
    </div>
    <div class="modal-input-item">
      <p class="label">征订状态：</p>
      <div style="width: 350px">
        <Select v-model="formData.status" placeholder="请选择产品状态">
          <Option v-for="item in statusList" :value="item.value" :key="item.value">{{ item.name }}</Option>
        </Select>
      </div>
    </div>
    <div class="modal-input-item">
      <p class="label">征订时间：</p>
      <div style="width: 350px">
        <DatePicker :value="[formData.startTime, formData.endTime]" type="datetimerange" placeholder="请选择征订起止时间" style="width: 100%" @on-change="timeChange"></DatePicker>
      </div>
    </div>
  </div>
</template>
<script>
  import { message } from '@/common/js/mixins'

  export default {
    props: {
      detail: {
        type: Object,
        default: () => {}
      },
      isWrite: {
        type: Boolean,
        default: false
      },
      codeList: {
        type: Array
      },
      goodsList: {
        type: Array
      }
    },
    data () {
      return {
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
        formData: {
          inviteCodeId: '',
          goodsId: '',
          status: '',
          startTime: '',
          endTime: ''
        }
      }
    },
    components: {},
    mixins: [message],
    created () {
      if (this.isWrite) {
        for (let k in this.formData) {
          this.formData[k] = this.detail[k]
        }
      }
    },
    methods: {
      timeChange (time) {
        this.formData.startTime = time[0]
        this.formData.endTime = time[1]
      },
      returnData () {
        if (this.formData.inviteCodeId === '') {
          this.warningInfo('请选择邀请码')
          return false
        }
        if (this.formData.goodsId === '') {
          this.warningInfo('请选择产品')
          return false
        }
        if (this.formData.status === '') {
          this.warningInfo('请选择状态')
          return false
        }
        if (this.formData.startTime === '') {
          this.warningInfo('请选择征订起止时间')
          return false
        }
        return {
          ...this.formData
        }
      }
    }
  }
</script>
<style lang="less" scope>
  .modal-input-item {
    margin-bottom: 15px;
  }
  .order-edit {
    overflow: auto;
  }
</style>
