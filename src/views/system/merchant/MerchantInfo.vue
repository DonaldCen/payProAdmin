<template>
  <a-modal
    v-model="show"
    :centered="true"
    :keyboard="false"
    :footer="null"
    :width="750"
    @cancel="handleCancleClick"
    title="用户信息">
    <a-layout class="user-info">
      <a-layout-sider class="user-info-side">
      </a-layout-sider>
      <a-layout-content class="user-content-one">
        <p>账户：{{userInfoData.username}}</p>
        <p :title="userInfoData.roleName">角色：{{userInfoData.roleName? userInfoData.roleName: '暂无角色'}}</p>
        <p>性别：{{sex}}</p>
        <p>电话：{{userInfoData.mobile ? userInfoData.mobile : '暂未绑定电话'}}</p>
        <p>邮箱：{{userInfoData.email ? userInfoData.email : '暂未绑定邮箱'}}</p>
      </a-layout-content>
      <a-layout-content class="user-content-two">
        <p>部门：{{userInfoData.deptName ? userInfoData.deptName : '暂无部门信息'}}</p>
        <p>
          状态：
          <template v-if="userInfoData.status === '0'">
            <a-tag color="red">锁定</a-tag>
          </template>
          <template v-else-if="userInfoData.status === '1'">
            <a-tag color="cyan">有效</a-tag>
          </template>
          <template v-else>
            {{userInfoData.status}}
          </template>
        </p>
        <p>创建时间：{{userInfoData.createTime}}</p>
        <p>最近登录：{{userInfoData.lastLoginTime}}</p>
        <p :title="userInfoData.description">描述：{{userInfoData.description}}</p>
      </a-layout-content>
    </a-layout>
  </a-modal>
</template>
<script>
export default {
  name: 'UserInfo',
  props: {
    userInfoVisiable: {
      require: true,
      default: false
    },
    userInfoData: {
      require: true
    }
  },
  computed: {
    show: {
      get: function () {
        return this.userInfoVisiable
      },
      set: function () {
      }
    },
    sex () {
      switch (this.userInfoData.ssex) {
        case '0':
          return '男'
        case '1':
          return '女'
        case '2':
          return '保密'
        default:
          return this.userInfoData.ssex
      }
    }
  },
  methods: {
    handleCancleClick () {
      this.$emit('close')
    }
  }
}
</script>
<style lang="less" scoped>
@import "MerchantInfo";
</style>
