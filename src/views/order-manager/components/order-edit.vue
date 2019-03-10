<template>
  <div class="order-edit">
    <div class="modal-input-item">
      <p class="label">邀请码：</p>
      <div style="width: 350px">
        <Input v-model="formData.inviteCode" :maxlength="10" placeholder="请输入邀请码（6-10位，由字母或数字组成）" :disabled="isWrite"/>
      </div>
    </div>
    <div class="modal-input-item">
      <p class="label">省：</p>
      <div style="width: 350px">
        <Input v-model="formData.province" :disabled="isWrite" placeholder="请输入省"/>
      </div>
    </div>
    <div class="modal-input-item">
      <p class="label">市：</p>
      <div style="width: 350px">
        <Input v-model="formData.city" :disabled="isWrite" placeholder="请输入市"/>
      </div>
    </div>
    <div class="modal-input-item">
      <p class="label">管理人员：</p>
      <div style="width: 350px">
        <Input v-model="formData.manageName"  placeholder="请输入管理人员"/>
      </div>
    </div>
    <div class="modal-input-item">
      <p class="label">注释：</p>
      <div style="width: 350px">
        <Input v-model="formData.other"  placeholder="请输入注释信息"/>
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
            inviteCode: '',
            province: '',
            city: '',
            manageName: '',
            other: ''
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
        if (!/^[0-9a-zA-Z]+$/.test(this.formData.inviteCode) || this.formData.inviteCode.length < 6) {
          this.warningInfo('邀请码不符合规范')
          return false
        }
        if (this.formData.province === '') {
          this.warningInfo('请输入省')
          return false
        }
        if (this.formData.city === '') {
          this.warningInfo('请输入市')
          return false
        }
        if (this.formData.manageName === '') {
          this.warningInfo('请输入管理人员')
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
