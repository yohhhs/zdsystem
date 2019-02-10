<template>
  <div class="order-edit">
    <div class="modal-input-item">
      <p class="label">邀请码</p>
      <div style="width: 350px">
        <Input v-model="formData.inviteCode" :maxlength="8" placeholder="请输入邀请码（4-8位，由大小写英文字母或数字组成）"/>
      </div>
    </div>
    <div class="modal-input-item">
      <p class="label">产品名称</p>
      <div style="width: 350px">
        <Input v-model="formData.goodsName" placeholder="请输入产品名称"/>
      </div>
    </div>
    <div class="modal-input-item">
      <p class="label">供应商</p>
      <div style="width: 350px">
        <Input v-model="formData.supplier" placeholder="请输入供应商"/>
      </div>
    </div>
    <div class="modal-input-item">
      <p class="label">产品单位</p>
      <div style="width: 350px">
        <Input v-model="formData.standard"  placeholder="请输入产品单位"/>
      </div>
    </div>
    <div class="modal-input-item">
      <p class="label">征订时间</p>
      <div style="width: 350px">
        <DatePicker :value="[this.formData.startTime, this.formData.endTime]" style="width: 100%" type="datetimerange" :editable="false" placeholder="请选择征订起止时间" @on-change="timeChange"></DatePicker>
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
      }
    },
    data () {
        return {
          formData: {
            goodsName: '',
            inviteCode: '',
            standard: '',
            supplier: '',
            startTime: '',
            endTime: '',
            goodsStatus: 1,
            goodsImage: '',
            salePrice: ''
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
        if (this.formData.inviteCode === '') {
          this.warningInfo('请输入邀请码')
          return false
        }
        if (!/^[0-9a-zA-Z]+$/.test(this.formData.inviteCode) || this.formData.inviteCode.length < 4) {
          this.warningInfo('邀请码不符合规范')
          return false
        }
        if (this.formData.goodsName === '') {
          this.warningInfo('请输入产品名称')
          return false
        }
        if (this.formData.supplier === '') {
          this.warningInfo('请输入供应商')
          return false
        }
        if (this.formData.standard === '') {
          this.warningInfo('请输入产品单位')
          return false
        }
        if (this.formData.startTime === '' || this.formData.endTime === '') {
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
