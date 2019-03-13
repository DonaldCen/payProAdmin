<template>
  <a-drawer
    title="修改通道名称"
    :maskClosable="false"
    width=650
    placement="right"
    :closable="false"
    @close="onClose"
    :visible="userEditVisiable"
    style="height: calc(100% - 55px);overflow: auto;padding-bottom: 53px;">
    <a-form :form="form">
      <a-form-item label='通道名称' v-bind="formItemLayout">
        <a-input readOnly v-decorator="['channelName']"/>
      </a-form-item>
      <a-form-item label='通道编码' v-bind="formItemLayout">
        <a-input readOnly v-decorator="['channelCode']"/>
      </a-form-item>
      <a-form-item label='通道成本利率' v-bind="formItemLayout">
        <a-input v-decorator="['chanelCostRate']"/>
      </a-form-item>
      <a-form-item label='通道成本费用' v-bind="formItemLayout">
        <a-input v-decorator="['channelCost']"/>
      </a-form-item>
      <a-form-item label='支付限额' v-bind="formItemLayout">
        <a-input v-decorator="['payLimit']"/>
      </a-form-item>
      <a-form-item label='状态' v-bind="formItemLayout">
        <a-radio-group
          v-decorator="[
            'status',
            {rules: [{ required: true, message: '请选择状态' }]}
          ]">
          <a-radio value="1">可用</a-radio>
          <a-radio value="0">禁用</a-radio>
        </a-radio-group>
      </a-form-item>
      <a-form-item label='结算类型' v-bind="formItemLayout">
        <a-radio-group
          v-decorator="[
            'settlementType',
            {rules: [{ required: true, message: '请选择类型' }]}
          ]">
          <a-radio value="0">D0</a-radio>
          <a-radio value="1">T+0</a-radio>
          <a-radio value="2">T+1</a-radio>
        </a-radio-group>
      </a-form-item>
    </a-form>
    <div class="drawer-bootom-button">
      <a-popconfirm title="确定放弃编辑？" @confirm="onClose" okText="确定" cancelText="取消">
        <a-button style="margin-right: .8rem">取消</a-button>
      </a-popconfirm>
      <a-button @click="handleSubmit" type="primary" :loading="loading">提交</a-button>
    </div>
  </a-drawer>
</template>
<script>
import {mapState} from 'vuex'

const formItemLayout = {
  labelCol: {span: 3},
  wrapperCol: {span: 18}
}
export default {
  name: 'ChannelEdit',
  props: {
    userEditVisiable: {
      default: false
    }
  },
  data () {
    return {
      formItemLayout,
      form: this.$form.createForm(this),
      deptTreeData: [],
      roleData: [],
      userDept: [],
      id: '',
      loading: false
    }
  },
  computed: {
    ...mapState({
      currentUser: state => state.account.user
    })
  },
  methods: {
    onClose () {
      this.loading = false
      this.form.resetFields()
      this.$emit('close')
    },
    handleSubmit () {
      this.form.validateFields((err, values) => {
        if (!err) {
          this.loading = true
          let channel = this.form.getFieldsValue()
          this.$put('channel', {
            ...channel
          }).then((r) => {
            this.loading = false
            this.$emit('success')
          }).catch(() => {
            this.loading = false
          })
        }
      })
    },
    setFormValues ({...channel}) {
      this.id = channel.id
      let fields = ['channelName', 'channelCode', 'chanelCostRate', 'channelCost', 'payLimit', 'status', 'settlementType']
      Object.keys(channel).forEach((key) => {
        if (fields.indexOf(key) !== -1) {
          this.form.getFieldDecorator(key)
          let obj = {}
          obj[key] = channel[key]
          this.form.setFieldsValue(obj)
        }
      })
    }
  }
}
</script>
