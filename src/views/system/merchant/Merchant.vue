<template>
  <a-card :bordered="false" class="card-area">
    <div :class="advanced ? 'search' : null">
      <!-- 搜索区域 -->
      <a-form layout="horizontal">
        <a-row >
        <div :class="advanced ? null: 'fold'">
            <a-col :md="12" :sm="24" >
              <a-form-item
                label="用户名"
                :labelCol="{span: 4}"
                :wrapperCol="{span: 18, offset: 2}">
                <a-input v-model="queryParams.username"/>
              </a-form-item>
            </a-col>
          <template v-if="advanced">
            <a-col :md="12" :sm="24" >
              <a-form-item
                label="创建时间"
                :labelCol="{span: 4}"
                :wrapperCol="{span: 18, offset: 2}">
                <range-date @change="handleDateChange" ref="createTime"></range-date>
              </a-form-item>
            </a-col>
          </template>
        </div>
          <span style="float: right; margin-top: 3px;">
            <a-button type="primary" @click="search">查询</a-button>
            <a-button style="margin-left: 8px" @click="reset">重置</a-button>
             <a @click="toggleAdvanced" style="margin-left: 8px">
              {{advanced ? '收起' : '展开'}}
              <a-icon :type="advanced ? 'up' : 'down'" />
            </a>
          </span>
        </a-row>
      </a-form>
    </div>
    <div>
      <div class="operator">
        <a-button type="primary" ghost @click="add" v-hasPermission="'merchantApply:add'">商户进件</a-button>
      </div>
      <!-- 表格区域 -->
      <a-table ref="TableInfo"
               :columns="columns"
               :dataSource="dataSource"
               :pagination="pagination"
               :loading="loading"
               :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
               :scroll="{ x: 2500 }"
               @change="handleTableChange">
        <template slot="operation" slot-scope="text, record">
          <a-icon v-hasPermission="'merchantApply:view'" type="eye" theme="twoTone" twoToneColor="#4a9ff5" @click="checkSignStatus(record)" title="查看签约状态"></a-icon>
          <a-badge v-hasNoPermission="'merchantApply:view'" status="warning" text="无权限"></a-badge>
        </template>
      </a-table>
    </div>
    <merchant-regist :merchantRegist="merchantRegist.visiable"
                     @close="handleMerchantRegistClose"
                     @success="handleMerchantApplyAddSuccess">
    </merchant-regist>
    <merchant-sign :merchantApplyVisiable="merchantApplyVisiable.visiable"
                   :merchantApplyData="merchantApplyVisiable.data"
                   @close="handleMerchantInfoClose"
                   @success="handleMerchantInfoSuccess"
    >

    </merchant-sign>
  </a-card>
</template>

<script>
import DeptInputTree from '../dept/DeptInputTree'
import RangeDate from '@/components/datetime/RangeDate'
import MerchantRegist from './MerchantRegist'
import MerchantSign from './MerchantInfo'

export default {
  name: 'Merchant',
  components: {MerchantSign, DeptInputTree, RangeDate, MerchantRegist},
  data () {
    return {
      advanced: false,
      userInfo: {
        visiable: false,
        data: {}
      },
      merchantApplyVisiable: {
        visiable: false,
        data: {}
      },
      merchantRegist: {
        visiable: false
      },
      signInfoVisible: {
        visiable: false
      },
      userAdd: {
        visiable: false
      },
      userEdit: {
        visiable: false
      },
      queryParams: {},
      filteredInfo: null,
      sortedInfo: null,
      paginationInfo: null,
      dataSource: [],
      selectedRowKeys: [],
      loading: false,
      pagination: {
        pageSizeOptions: ['10', '20', '30', '40', '100'],
        defaultCurrent: 1,
        defaultPageSize: 10,
        showQuickJumper: true,
        showSizeChanger: true,
        showTotal: (total, range) => `显示 ${range[0]} ~ ${range[1]} 条记录，共 ${total} 条记录`
      }
    }
  },
  computed: {
    columns () {
      let { sortedInfo } = this
      sortedInfo = sortedInfo || {}
      return [{
        title: 'ID',
        dataIndex: 'id',
        sorter: true,
        sortOrder: sortedInfo.columnKey === 'id' && sortedInfo.order
      }, {
        title: '身份证姓名',
        dataIndex: 'idCardName',
        sorter: true,
        sortOrder: sortedInfo.columnKey === 'idCardName' && sortedInfo.order
      }, {
        title: '身份证号码',
        dataIndex: 'idCardNumber',
        sorter: true,
        sortOrder: sortedInfo.columnKey === 'idCardNumber' && sortedInfo.order
      }, {
        title: '身份证有效期限',
        dataIndex: 'idCardValidTime',
        sorter: true,
        sortOrder: sortedInfo.columnKey === 'idCardValidTime' && sortedInfo.order
      }, {
        title: '开户名称',
        dataIndex: 'accountName',
        sorter: true,
        sortOrder: sortedInfo.columnKey === 'accountName' && sortedInfo.order
      }, {
        title: '开户银行',
        dataIndex: 'accountBank',
        sorter: true,
        sortOrder: sortedInfo.columnKey === 'accountBank' && sortedInfo.order
      }, {
        title: '开户银行全称',
        dataIndex: 'bankName',
        sorter: true,
        sortOrder: sortedInfo.columnKey === 'bankName' && sortedInfo.order
      }, {
        title: '银行账号',
        dataIndex: 'accountNumber',
        sorter: true,
        sortOrder: sortedInfo.columnKey === 'accountNumber' && sortedInfo.order
      }, {
        title: '门店街道名称',
        dataIndex: 'storeStreet',
        sorter: true,
        sortOrder: sortedInfo.columnKey === 'storeStreet' && sortedInfo.order
      }, {
        title: '商户简称',
        dataIndex: 'merchantShortName',
        sorter: true,
        sortOrder: sortedInfo.columnKey === 'merchantShortName' && sortedInfo.order
      }, {
        title: '客服电话',
        dataIndex: 'servicePhone',
        sorter: true,
        sortOrder: sortedInfo.columnKey === 'servicePhone' && sortedInfo.order
      }, {
        title: '售卖商品/提供服务描述',
        dataIndex: 'productDesc',
        sorter: true,
        sortOrder: sortedInfo.columnKey === 'productDesc' && sortedInfo.order
      }, {
        title: '费率',
        dataIndex: 'rate',
        sorter: true,
        sortOrder: sortedInfo.columnKey === 'rate' && sortedInfo.order
      }, {
        title: '联系人姓名',
        dataIndex: 'contact',
        sorter: true,
        sortOrder: sortedInfo.columnKey === 'contact' && sortedInfo.order
      }, {
        title: '手机号码',
        dataIndex: 'servicePhone',
        sorter: true,
        sortOrder: sortedInfo.columnKey === 'servicePhone' && sortedInfo.order
      }, {
        title: '状态',
        dataIndex: 'status',
        customRender: (text, row, index) => {
          switch (text) {
            case 'AUDITING':
              return <a-tag color="cyan">审核中</a-tag>
            case 'REJECTED':
              return <a-tag color="red">已驳回</a-tag>
            case 'FROZEN':
              return <a-tag color="blue">已冻结</a-tag>
            case 'TO_BE_SIGNED':
              return <a-tag color="green">待签约</a-tag>
            case 'FINISH':
              return <a-tag color="orange">完成</a-tag>
            default:
              return <a-tag color="#2db7f5">其他</a-tag>
          }
        },
        filters: [
          { text: '审核中', value: 'AUDITING' },
          { text: '已驳回', value: 'REJECTED' },
          { text: '已冻结', value: 'FROZEN' },
          { text: '待签约', value: 'TO_BE_SIGNED' },
          { text: '完成', value: 'FINISH' }
        ],
        filterMultiple: false,
        onFilter: (value, record) => record.status.includes(value)
      }, {
        title: '联系邮箱',
        dataIndex: 'contactEmail',
        sorter: true,
        sortOrder: sortedInfo.columnKey === 'contactEmail' && sortedInfo.order
      }, {
        title: '操作',
        dataIndex: 'operation',
        scopedSlots: { customRender: 'operation' }
      }]
    }
  },
  mounted () {
    this.fetch()
  },
  methods: {
    onSelectChange (selectedRowKeys) {
      this.selectedRowKeys = selectedRowKeys
    },
    toggleAdvanced () {
      this.advanced = !this.advanced
      if (!this.advanced) {
        this.queryParams.createTimeFrom = ''
        this.queryParams.createTimeTo = ''
      }
    },
    view (record) {
      this.userInfo.data = record
      this.userInfo.visiable = true
    },
    add () {
      this.merchantRegist.visiable = true
    },
    handleUserAddClose () {
      this.userAdd.visiable = false
    },
    handleMerchantApplyAddSuccess () {
      this.merchantRegist.visiable = false
      this.$message.success('生成二维码中。。')
      this.search()
    },
    handleMerchantInfoSuccess () {
      this.merchantApplyVisiable.visiable = false
      this.search()
    },
    edit (record) {
      this.$refs.userEdit.setFormValues(record)
      this.userEdit.visiable = true
    },
    checkSignStatus (record) {
      this.merchantApplyVisiable.data = record
      this.merchantApplyVisiable.visiable = true
    },
    handleUserEditClose () {
      this.userEdit.visiable = false
    },
    handleUserEditSuccess () {
      this.userEdit.visiable = false
      this.$message.success('修改用户成功')
      this.search()
    },
    handleUserInfoClose () {
      this.userInfo.visiable = false
    },
    handleMerchantRegistClose () {
      this.merchantRegist.visiable = false
    },
    handleMerchantInfoClose () {
      this.merchantApplyVisiable.visiable = false
    },
    handleDeptChange (value) {
      this.queryParams.deptId = value || ''
    },
    handleDateChange (value) {
      if (value) {
        this.queryParams.createTimeFrom = value[0]
        this.queryParams.createTimeTo = value[1]
      }
    },
    batchDelete () {
      this.$message.warning('请选择需要删除的记录')
    },
    resetPassword () {
      if (!this.selectedRowKeys.length) {
        this.$message.warning('请选择需要重置密码的用户')
        return
      }
      let that = this
      this.$confirm({
        title: '确定重置选中用户密码?',
        content: '当您点击确定按钮后，这些用户的密码将会重置为1234qwer',
        centered: true,
        onOk () {
          let usernames = []
          for (let key of that.selectedRowKeys) {
            usernames.push(that.dataSource[key].username)
          }
          that.$put('user/password/reset', {
            usernames: usernames.join(',')
          }).then(() => {
            that.$message.success('重置用户密码成功')
            that.selectedRowKeys = []
          })
        },
        onCancel () {
          that.selectedRowKeys = []
        }
      })
    },
    exportExcel () {
      let {sortedInfo, filteredInfo} = this
      let sortField, sortOrder
      // 获取当前列的排序和列的过滤规则
      if (sortedInfo) {
        sortField = sortedInfo.field
        sortOrder = sortedInfo.order
      }
      this.$export('user/excel', {
        sortField: sortField,
        sortOrder: sortOrder,
        ...this.queryParams,
        ...filteredInfo
      })
    },
    search () {
      let {sortedInfo, filteredInfo} = this
      let sortField, sortOrder
      // 获取当前列的排序和列的过滤规则
      if (sortedInfo) {
        sortField = sortedInfo.field
        sortOrder = sortedInfo.order
      }
      this.fetch({
        sortField: sortField,
        sortOrder: sortOrder,
        ...this.queryParams,
        ...filteredInfo
      })
    },
    reset () {
      // 取消选中
      this.selectedRowKeys = []
      // 重置分页
      this.$refs.TableInfo.pagination.current = this.pagination.defaultCurrent
      if (this.paginationInfo) {
        this.paginationInfo.current = this.pagination.defaultCurrent
        this.paginationInfo.pageSize = this.pagination.defaultPageSize
      }
      // 重置列过滤器规则
      this.filteredInfo = null
      // 重置列排序规则
      this.sortedInfo = null
      // 重置查询参数
      this.queryParams = {}
      // 清空部门树选择
      this.$refs.deptTree.reset()
      // 清空时间选择
      if (this.advanced) {
        this.$refs.createTime.reset()
      }
      this.fetch()
    },
    handleTableChange (pagination, filters, sorter) {
      // 将这三个参数赋值给Vue data，用于后续使用
      this.paginationInfo = pagination
      this.filteredInfo = filters
      this.sortedInfo = sorter

      this.userInfo.visiable = false
      this.fetch({
        sortField: sorter.field,
        sortOrder: sorter.order,
        ...this.queryParams,
        ...filters
      })
    },
    fetch (params = {}) {
      // 显示loading
      this.loading = true
      if (this.paginationInfo) {
        // 如果分页信息不为空，则设置表格当前第几页，每页条数，并设置查询分页参数
        this.$refs.TableInfo.pagination.current = this.paginationInfo.current
        this.$refs.TableInfo.pagination.pageSize = this.paginationInfo.pageSize
        params.pageSize = this.paginationInfo.pageSize
        params.pageNum = this.paginationInfo.current
      } else {
        // 如果分页信息为空，则设置为默认值
        params.pageSize = this.pagination.defaultPageSize
        params.pageNum = this.pagination.defaultCurrent
      }
      this.$get('merchantApply', {
        ...params
      }).then((r) => {
        let data = r.data
        const pagination = { ...this.pagination }
        pagination.total = data.total
        this.dataSource = data.rows
        this.pagination = pagination
        // 数据加载完毕，关闭loading
        this.loading = false
      })
    }
  }
}
</script>
<style lang="less" scoped>
@import "../../../../static/less/Common";
</style>
