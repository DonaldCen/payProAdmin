<template>
  <a-drawer
    title="新增通道"
    :maskClosable="false"
    width=650
    placement="right"
    :closable="false"
    @close="onClose"
    :visible="userAddVisiable"
    style="height: calc(100% - 55px);overflow: auto;padding-bottom: 53px;">
    <a-form :form="form">
      <a-form-item label='通道名称'
                   v-bind="formItemLayout"
                   :validateStatus="validateStatus"
                   :help="help">
        <a-input v-model="channel.channelName"
                 @blur="handleChannelNameBlur"
                 v-decorator="['channelName',{rules: [{ required: true, message: '通道名称不能为空'}]}]"/>
      </a-form-item>
      <a-form-item label='通道编码'
                   v-bind="formItemLayout"
                   :validateStatus="validateStatus"
                   :help="help">
        <a-input v-model="channel.channelCode"
                 v-decorator="['channelCode',{rules: [{ required: true, message: '通道编码不能为空'}]}]"/>
      </a-form-item>
      <a-form-item label='通道成本利率'
                   v-bind="formItemLayout"
                   :validateStatus="validateStatus"
                   :help="help">
        <a-input v-model="channel.chanelCostRate"
                 v-decorator="['channelCode',{rules: [{ required: true, message: '通道成本利率不能为空'}]}]"/>
      </a-form-item>
      <a-form-item label='通道成本费用'
                   v-bind="formItemLayout"
                   :validateStatus="validateStatus"
                   :help="help">
        <a-input v-model="channel.channelCost"/>
      </a-form-item>
      <a-form-item label='支付限额'
                   v-bind="formItemLayout"
                   :validateStatus="validateStatus"
                   :help="help">
        <a-input v-model="channel.payLimit"/>
      </a-form-item>
      <a-form-item label='状态' v-bind="formItemLayout">
        <a-radio-group
          v-model="channel.status"
          v-decorator="['status',{rules: [{ required: true, message: '请选择状态'}]}]">
          <a-radio value="0">锁定</a-radio>
          <a-radio value="1">有效</a-radio>
        </a-radio-group>
      </a-form-item>
      <a-form-item label='结算类型' v-bind="formItemLayout">
        <a-radio-group
          v-model="channel.settlementType"
          v-decorator="['settlementType',{rules: [{ required: true, message: '请选择类型'}]}]">
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
const formItemLayout = {
  labelCol: {span: 5},
  wrapperCol: {span: 18}
}
export default {
  name: 'ChannelAdd',
  props: {
    userAddVisiable: {
      default: false
    }
  },
  data () {
    return {
      channel: {
        channelName: ''
      },
      loading: false,
      roleData: [],
      deptTreeData: [],
      formItemLayout,
      form: this.$form.createForm(this),
      validateStatus: '',
      help: ''
    }
  },
  methods: {
    reset () {
      this.validateStatus = ''
      this.help = ''
      this.channel.channelName = ''
      this.loading = false
      this.form.resetFields()
    },
    onClose () {
      this.reset()
      this.$emit('close')
    },
    handleSubmit () {
      if (this.validateStatus !== 'success') {
        this.handleChannelNameBlur()
      }
      this.form.validateFields((err, values) => {
        if (!err && this.validateStatus === 'success') {
          this.loading = true
          this.$post('channel', {
            ...this.channel
          }).then((r) => {
            this.reset()
            this.$emit('success')
          }).catch(() => {
            this.loading = false
          })
        }
      })
    },
    handleChannelNameBlur () {
      let channelName = this.channel.channelName.trim()
      if (channelName.length) {
        this.validateStatus = 'validating'
        this.$get(`channel/check/${channelName}`).then((r) => {
          if (r.data) {
            this.validateStatus = 'success'
            this.help = ''
          } else {
            this.validateStatus = 'error'
            this.help = '抱歉，该通道名称已存在'
          }
        })
      } else {
        this.validateStatus = 'error'
        this.help = '通道名称不能为空'
      }
    }
  }
}
</script>
