<template>
  <a-modal
    v-model="show"
    centered
    style="top: 20px;"
    @cancel="handleCancleClick"
    @ok="handleOk"
    title="商户签约状态">
    <a-layout class="user-info">
      <a-layout-sider class="user-info-side">
      </a-layout-sider>
      <a-layout-content class="user-content-one">
        <div v-show="qrCodeUrl" class="response" v-if="merchantSignStatus === 'TO_BE_SIGNED'">
          <p>当前入驻申请已通过审核</p>
          <p>请
            <a-tag color="green">{{merchantApplyData.merchantShortName}}</a-tag>
            微信扫码完成签约
          </p>
          <qrcode :value="qrCodeUrl" v-if="qrCodeUrl" :options="{ size: 170 }"></qrcode>
        </div>
        <div v-if="merchantSignStatus === 'FINISH'">
          <p>签约已完成！</p>
        </div>
        <p>申请单号：{{merchantApplyData.businessCode}}</p>
        <p>商户名称：{{merchantApplyData.merchantShortName}}</p>
        <p>门店名称：{{merchantApplyData.storeName}}</p>
        <p>结算银行卡：{{merchantApplyData.accountNumber}}</p>
        <p>
          签约状态：
          <template v-if="merchantSignStatus === 'AUDITING'">
            <a-tag color="cyan">审核中</a-tag>
          </template>
          <template v-else-if="merchantSignStatus === 'REJECTED'">
            <a-tag color="red">已驳回</a-tag>
          </template>
          <template v-else-if="merchantSignStatus === 'FROZEN'">
            <a-tag color="blue">已冻结</a-tag>
          </template>
          <template v-else-if="merchantSignStatus === 'TO_BE_SIGNED'">
            <a-tag color="green">待签约</a-tag>
          </template>
          <template v-else-if="merchantSignStatus === 'FINISH'">
            <a-tag color="orange">完成</a-tag>
          </template>
          <template v-else>
            <a-tag color="#2db7f5">其他</a-tag>
          </template>
        </p>
        <div v-if="merchantSignStatus === 'REJECTED'">
          <p>签约已完成！</p>
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
      merchantSignStatus: 'AUDITING',
      auditDetail: ''
    }
  },
  computed: {
    show: {
      get: function () {
        if (this.merchantApplyVisiable) {
          this.queryStausByApplymentId()
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
      let applymentId = this.merchantApplyData.applymentID
      let merchantSignStatus = this.merchantApplyData.status
      if (applymentId) {
        let apply = {
          'applymentId': applymentId
        }
        if (merchantSignStatus === 'FINISH') {
          this.$post('merchantSign/upStatus', {
            ...apply
          }).then((r) => {
            let code = r.data.code
            let msg = r.data.msg
            if (code !== 0) {
              this.$notification['warn']({
                message: '提示',
                description: msg
              })
            }
          }).catch(() => {
          })
        }
      }
      this.$emit('success')
    },
    queryStausByApplymentId () {
      let applymentId = this.merchantApplyData.applymentID
      let merchantSignStatus = this.merchantApplyData.status
      if (applymentId) {
        let apply = {
          'applymentId': applymentId
        }
        this.$post('merchantSign/apply', {
          ...apply
        }).then((r) => {
          let code = r.data.code
          let data = r.data.data
          let msg = r.data.msg
          if (code === 0) {
            let applymentState = data.applymentState
            if (applymentState === 'TO_BE_SIGNED' || applymentState === 'FINISH') {
              this.qrCodeUrl = data.signUrl
            } else if (applymentState === 'REJECTED') {
              let auditDetail = data.auditDetail
              this.auditDetail = auditDetail
            }
            merchantSignStatus = applymentState
          } else {
            this.$notification['warn']({
              message: '提示',
              description: msg
            })
          }
        }).catch(() => {
        })
      }
      this.merchantSignStatus = merchantSignStatus
    }
  }
}
</script>
<style lang="less" scoped>
@import "MerchantInfo";
</style>
