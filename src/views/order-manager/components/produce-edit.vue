<template>
  <div class="produce-edit">
    <div class="modal-input-item">
      <p class="label">产品编码：</p>
      <div style="width: 350px">
        <Input v-model="formData.goodsCode" :maxlength="10" placeholder="请输入产品编码" :disabled="isWrite"/>
      </div>
    </div>
    <div class="modal-input-item">
      <p class="label">产品名称：</p>
      <div style="width: 350px">
        <Input v-model="formData.goodsName" placeholder="请输入产品名称"/>
      </div>
    </div>
    <div class="modal-input-item">
      <p class="label">供应商：</p>
      <div style="width: 350px">
        <Input v-model="formData.supplier" placeholder="请输入供应商"/>
      </div>
    </div>
    <div class="modal-input-item">
      <p class="label">产品单位：</p>
      <div style="width: 350px">
        <Input v-model="formData.standard"  placeholder="请输入单位"/>
      </div>
    </div>
    <div class="modal-input-item">
      <p class="label">产品状态：</p>
      <div style="width: 350px">
        <Select v-model="formData.status" placeholder="请选择产品状态">
          <Option v-for="item in statusList" :value="item.value" :key="item.value">{{ item.name }}</Option>
        </Select>
      </div>
    </div>
    <div class="modal-input-item">
      <p class="label">产品图片：</p>
      <div style="width: 350px">
        <Upload
          ref="listUpload"
          withCredentials
          paste
          :show-upload-list="false"
          name="files"
          accept="image/*"
          :format="['jpg','jpeg','png']"
          :on-success="imgListSuccess"
          action="https://www.topasst.com/solicitCms/file/uploadFile">
          <!--<Button type="primary" ghost>上传产品图片</Button>-->
          <div class="img-wrapper">
            <span v-if="formData.goodsImage === ''" class="upload-tip">+</span>
            <img v-else :src="formData.goodsImage">
          </div>
        </Upload>
      </div>
    </div>
  </div>
</template>

<script>
  import { message } from '@/common/js/mixins'

  export default {
    name: 'produce-edit',
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
    data() {
      return {
        formData: {
          goodsCode: '',
          goodsName: '',
          supplier: '',
          standard: '',
          status: '',
          goodsImage: ''
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
        ]
      }
    },
    components: {},
    mixins: [message],
    created() {
      if (this.isWrite) {
        for(let k in this.formData) {
          this.formData[k] = this.detail[k]
        }
      }
    },
    methods: {
      imgListSuccess (data) {
        this.formData.goodsImage = 'https://www.topasst.com/images/' + data.data
      },
      returnData () {
        if (this.formData.goodsCode === '') {
          this.warningInfo('请输入产品编码')
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
        if (this.formData.status === '') {
          this.warningInfo('请选择产品状态')
          return false
        }
        if (this.formData.goodsImage === '') {
          this.warningInfo('请上传产品图片')
          return false
        }
        return {
          ...this.formData
        }
      }
    }
  }
</script>

<style lang="less" scoped>
  .modal-input-item {
    margin-bottom: 15px;
    .label {
      padding-right: 10px;
      text-align: right;
    }
  }
  .order-edit {
    overflow: auto;
  }
  .img-wrapper {
    width: 100px;
    height: 100px;
    background-color: #eee;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    img {
      max-width: 100%;
      max-height: 100%;
    }
    .upload-tip {
      font-size: 60px;
      color: #ddd;
    }
  }
</style>
