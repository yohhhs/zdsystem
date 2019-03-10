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
        <Input v-model="formData.goodsName" :disabled="isWrite" placeholder="请输入产品名称"/>
      </div>
    </div>
    <div class="modal-input-item">
      <p class="label">供应商：</p>
      <div style="width: 350px">
        <Input v-model="formData.supplier" :disabled="isWrite" placeholder="请输入供应商"/>
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
          v-show="!isLook"
          ref="listUpload"
          withCredentials
          name="files"
          accept="image/*"
          :format="['jpg','jpeg','png']"
          :on-success="imgListSuccess"
          action="https://www.topasst.com/cms/file/uploadFile">
          <Button type="primary" ghost>上传产品图片</Button>
        </Upload>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'produce-edit',
    props: {},
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
    mixins: [],
    created() {
    },
    methods: {
      imgListSuccess (data) {
        this.formData.goodsImage = data.data
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
</style>
