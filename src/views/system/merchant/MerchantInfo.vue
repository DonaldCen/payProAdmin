<template>
  <a-modal
    v-model="show"
    centered
    style="top: 20px;"
    @cancel="handleCancleClick"
    @ok="handleOk"
    title="商户签约状态">
    <a-layout class="user-info">
      <a-layout-content class="user-content-one">
        <div v-show="merchantApplyData.signUrl" class="response" v-if="merchantApplyData.status === 'TO_BE_SIGNED'">
          <p>当前入驻申请已通过审核</p>
          <p>请
            <a-tag color="green">{{merchantApplyData.merchantShortName}}</a-tag>
            微信扫码完成签约
          </p>
          <qrcode :value="merchantApplyData.signUrl" v-if="merchantApplyData.signUrl" :options="{ size: 170 }"></qrcode>
        </div>
        <div v-if="merchantApplyData.status === 'FINISH'">
          <p>签约已完成！</p>
        </div>
        <p>申请单号：{{merchantApplyData.businessCode}}</p>
        <p>商户名称：{{merchantApplyData.merchantShortName}}</p>
        <p>门店名称：{{merchantApplyData.storeName}}</p>
        <p>结算银行卡：{{merchantApplyData.accountNumber}}</p>
        <p>
          签约状态：
          <template v-if="merchantApplyData.status === 'AUDITING'">
            <a-tag color="cyan">审核中</a-tag>
          </template>
          <template v-else-if="merchantApplyData.status === 'REJECTED'">
            <a-tag color="red">已驳回</a-tag>
          </template>
          <template v-else-if="merchantApplyData.status === 'FROZEN'">
            <a-tag color="blue">已冻结</a-tag>
          </template>
          <template v-else-if="merchantApplyData.status === 'TO_BE_SIGNED'">
            <a-tag color="green">待签约</a-tag>
          </template>
          <template v-else-if="merchantApplyData.status === 'FINISH'">
            <a-tag color="orange">完成</a-tag>
          </template>
          <template v-else>
            <a-tag color="#2db7f5">{{merchantApplyData.status}}</a-tag>
          </template>
        </p>
        <div v-if="merchantApplyData.apply_desc != null">
            <p>提示：{{merchantApplyData.apply_desc}}</p>
        </div>
      </a-layout-content>
    </a-layout>
  </a-modal>
</template>
<script>
import QRcode from '@xkeshi/vue-qrcode'
export default {
  name: 'MerchantSign',
  components: {qrcode: QRcode},
  props: {
    merchantApplyVisiable: {
      require: true,
      default: false
    },
    merchantApplyData: {
      require: true
    }
  },
  data () {
    return {
      qrCodeUrl: 'https://pay.weixin.qq.com',
      auditDetail: '',
      merchantSignStatus: ''
    }
  },
  computed: {
    show: {
      get: function () {
        if (this.merchantApplyVisiable) {
        }
        return this.merchantApplyVisiable
      },
      set: function () {
      }
    }
  },
  methods: {
    handleCancleClick () {
      this.$emit('close')
    },
    handleOk () {
      this.$emit('success')
    }
  }
}
</script>
<style lang="less" scoped>
@import "MerchantInfo";
</style>
