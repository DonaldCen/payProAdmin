<template>
  <a-drawer
    title="商户进件"
    :maskClosable="false"
    width=650
    placement="right"
    :closable="false"
    @close="onClose"
    :visible="userAddVisiable"
    style="height: calc(100% - 55px);overflow: auto;padding-bottom: 53px;">
    <a-form-item label='角色' v-bind="formItemLayout">
      <a-select
        mode="multiple"
        :allowClear="true"
        v-model="bank.id"
        style="width: 100%"
        v-decorator="['bankId',{rules: [{ required: true, message: '请选择角色' }]}]">
        <a-select-option v-for="r in roleData" :key="r.id">{{r.bankName}}</a-select-option>
      </a-select>
    </a-form-item>
      <div class="drawer-bootom-button">
        <a-popconfirm title="确定放弃编辑？" @confirm="onClose" okText="确定" cancelText="取消">
          <a-button style="margin-right: .8rem">取消</a-button>
        </a-popconfirm>
        <a-button @click="handleSubmit" type="primary" :loading="loading">提交</a-button>
      </div>
  </a-drawer>
</template>
<script>
const formItemLayout = {
  labelCol: { span: 4 },
  wrapperCol: { span: 18 }
}
export default {
  name: 'MerchantRegist',
  props: {
    userAddVisiable: {
      default: false
    }
  },
  data () {
    return {
      bank: {},
      parentId: 0,
      loading: false,
      roleData: [],
      deptTreeData: [],
      formItemLayout,
      defaultPassword: '1234qwer',
      form: this.$form.createForm(this),
      validateStatus: '',
      help: ''
    }
  },
  methods: {
    handleItemChange (val) {
      console.log('active item:', val)
      this.parentId = val
    }
  },
  watch: {
    userAddVisiable () {
    }
  }
}
</script>
