<template>
  <a-drawer
    title="商户进件"
    :maskClosable="false"
    width=650
    placement="right"
    :closable="false"
    @close="onClose"
    :visible="merchantRegist"
    style="height: calc(100% - 55px);overflow: auto;padding-bottom: 53px;">
    <a-form :form="form">
      <a-form-item label='身份证正面照' v-bind="formItemLayout">
        <div class="option-area">
          <a-upload :fileList="idCardFrontFileList"
                    class="upload-area"
                    :remove="idCardFrontHandleRemove"
                    :disabled="idCardFrontFileList.length === 1"
                    :beforeUpload="idCardFrontBeforeUpload"
                    v-decorator="['id_card_copy',{rules: [{ required: true, message: '身份证正面照不能为空'}]}]"
          >
            <a-button>
              <a-icon type="upload" /> Upload
            </a-button>
          </a-upload>
          <div class="button-area">
            <a-button
              @click="idCardFrontHandleUpload"
              :disabled="idCardFrontFileList.length === 0"
              :loading="idCardFrontUploading">
              {{idCardFrontUploading ? '导入中' : '导入图片' }}
            </a-button>
          </div>
        </div>
      </a-form-item>
      <a-form-item label='身份证背面照' v-bind="formItemLayout">
        <div class="option-area">
          <a-upload :fileList="idCardBackFileList"
                    class="upload-area"
                    :remove="idCardBackHandleRemove"
                    :disabled="idCardBackFileList.length === 1"
                    :beforeUpload="idCardBackBeforeUpload"
                    v-decorator="['id_card_national',{rules: [{ required: true, message: '身份证背面照不能为空'}]}]"
          >
            <a-button>
              <a-icon type="upload" /> Upload
            </a-button>
          </a-upload>
          <div class="button-area">
            <a-button
              @click="idCardBackHandleUpload"
              :disabled="idCardBackFileList.length === 0"
              :loading="idCardBackUploading">
              {{idCardBackUploading ? '导入中' : '导入图片' }}
            </a-button>
          </div>
        </div>
      </a-form-item>
      <a-form-item label='身份证姓名'
                   v-bind="formItemLayout">
        <a-input v-model="resultData.id_card_name"
                 v-decorator="['id_card_name',{rules: [{ required: true, message: '身份证姓名不能为空'}]}]"/>
      </a-form-item>
      <a-form-item label='身份证号码'
                   v-bind="formItemLayout">
        <a-input v-model="resultData.id_card_number"
                 v-decorator="['id_card_number',{rules: [{ required: true, message: '身份证号码不能为空'}]}]"/>
      </a-form-item>
      <a-form-item label='身份证有效时间'
                   v-bind="formItemLayout">
        <a-range-picker @change="idCardDateOnChange" format="YYYY-MM-DD"
                        v-decorator="['id_card_valid_time',{rules: [{ required: true, message: '身份证有效时间不能为空'}]}]"/>
      </a-form-item>
      <a-form-item label='开户名称'
                   v-bind="formItemLayout">
        <a-input v-model="resultData.account_name"
                 v-decorator="['account_name',{rules: [{ required: true, message: '开户名称不能为空'}]}]"/>
      </a-form-item>
      <a-form-item label='开户银行' v-bind="formItemLayout">
        <a-select
          @change="handleBankChange"
          style="width: 400px"
          v-decorator="['bankId',{rules: [{ required: true, message: '请选择开户银行' }]}]">
          <a-select-option v-for="bank in bankData" :key="bank.id">{{bank.bankName}}</a-select-option>
        </a-select>
      </a-form-item>
      <a-form-item label='银行省市' v-bind="formItemLayout">
        <a-cascader :options="bankArea" style="width: 400px" @change="onChangeBankArea" :loadData="loadBankAreaData"
                    placeholder="银行省市" changeOnSelect  v-decorator="['bank_address_code',{rules: [{ required: true, message: '请选择银行省市' }]}]"/>
      </a-form-item>
      <a-form-item label='银行全称' v-bind="formItemLayout">
        <a-auto-complete
          style="width: 400px"
          @search="handleBranchSearch"
          placeholder="请输入详细支行信息"
          v-model="resultData.bank_name"
          v-decorator="['bank_name',{rules: [{ required: true, message: '详细支行信息不能为空'}]}]"
          @select="handleSubBankSelect">
          <template slot="dataSource">
            <a-select-option v-for="subBank in subBranch" :key="subBank.id.toString()">
              {{subBank.bankName}}
            </a-select-option>
          </template>
        </a-auto-complete>
      </a-form-item>
      <a-form-item label='银行账户'
                   v-bind="formItemLayout">
        <a-input v-model="resultData.account_number"
                 v-decorator="['account_number',{rules: [{ required: true, message: '银行账户不能为空'}]}]"/>
      </a-form-item>
      <a-form-item label='门店名称'
                   v-bind="formItemLayout">
        <a-input v-model="resultData.store_name"
                 v-decorator="['store_name',{rules: [{ required: true, message: '门店名称不能为空'}]}]"/>
      </a-form-item>
      <a-form-item label='门店省市' v-bind="formItemLayout">
        <a-cascader :options="storeArea" style="width: 400px"
                    @change="onChangeStoreArea"
                    :loadData="loadBankStoreData"
                    placeholder="门店省市" changeOnSelect
                    v-decorator="['store_address_code',{rules: [{ required: true, message: '门店省市不能为空'}]}]"
        />
      </a-form-item>
      <a-form-item label='街道名称'
                   v-bind="formItemLayout">
        <a-input v-model="resultData.store_street"
                 v-decorator="['store_street',{rules: [{ required: true, message: '街道名称不能为空'}]}]"/>
      </a-form-item>
      <a-form-item label='门店门口照' v-bind="formItemLayout">
        <div class="option-area">
          <a-upload :fileList="storeInFileList"
                    class="upload-area"
                    :remove="storeInHandleRemove"
                    :disabled="storeInFileList.length === 1"
                    :beforeUpload="storeInBeforeUpload"
                    v-decorator="['store_entrance_pic',{rules: [{ required: true, message: '门店门口照不能为空'}]}]"
          >
            <a-button>
              <a-icon type="upload" /> Upload
            </a-button>
          </a-upload>
          <div class="button-area">
            <a-button
              @click="storeInHandleUpload"
              :disabled="storeInFileList.length === 0"
              :loading="storeInUploading">
              {{storeInUploading ? '导入中' : '导入图片' }}
            </a-button>
          </div>
        </div>
      </a-form-item>
      <a-form-item label='店内环境照' v-bind="formItemLayout">
        <div class="option-area">
          <a-upload :fileList="storeOutFileList"
                    class="upload-area"
                    :remove="storeOutHandleRemove"
                    :disabled="storeOutFileList.length === 1"
                    :beforeUpload="storeOutBeforeUpload"
                    v-decorator="['indoor_pic',{rules: [{ required: true, message: '店内环境照不能为空'}]}]"
          >
            <a-button>
              <a-icon type="upload" /> Upload
            </a-button>
          </a-upload>
          <div class="button-area">
            <a-button
              @click="storeOutHandleUpload"
              :disabled="storeOutFileList.length === 0"
              :loading="storeOutUploading">
              {{storeOutUploading ? '导入中' : '导入图片' }}
            </a-button>
          </div>
        </div>
      </a-form-item>
      <a-form-item label='商户简称'
                   v-bind="formItemLayout">
        <a-input v-model="resultData.merchant_shortname"
                 v-decorator="['merchant_shortname',{rules: [{ required: true, message: '商户简称不能为空'}]}]"/>
      </a-form-item>
      <a-form-item label='客服电话'
                   v-bind="formItemLayout">
        <a-input v-model="resultData.service_phone"
                 v-decorator="['service_phone',{rules: [{ required: true, message: '客服电话不能为空'}]}]"/>
      </a-form-item>
      <a-form-item label='商户描述'
                   v-bind="formItemLayout">
        <a-select
          @change="handleProductDescChange"
          style="width: 400px"
          v-decorator="['product_desc',{rules: [{ required: true, message: '客服电话不能为空'}]}]"
        >
          <a-select-option v-for="desc in productDescData" :key="desc">{{desc}}</a-select-option>
        </a-select>
      </a-form-item>
      <a-form-item label='费率' v-bind="formItemLayout">
        <a-select
          @change="handleRateChange"
          style="width: 400px"
          v-decorator="['rateId',{rules: [{ required: true, message: '请选择费率' }]}]">
          <a-select-option v-for="rate in rateData" :key="rate.id">{{rate.rate}}</a-select-option>
        </a-select>
      </a-form-item>
      <a-form-item label='联系人姓名'
                   v-bind="formItemLayout">
        <a-input v-model="resultData.contact"
                 v-decorator="['contact',{rules: [{ required: true, message: '联系人姓名不能为空'}]}]"/>
      </a-form-item>
      <a-form-item label='手机号'
                   v-bind="formItemLayout">
        <a-input v-model="resultData.contact_phone"
                 v-decorator="['contact_phone',{rules: [{ required: true, message: '手机号不能为空'}]}]"/>
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
  labelCol: { span: 6 },
  wrapperCol: { span: 18 }
}
export default {
  name: 'MerchantRegist',
  props: {
    merchantRegist: {
      default: false
    }
  },
  data () {
    return {
      resultData: {},
      bankData: [],
      rateData: [],
      productDescData: [],
      bankArea: [],
      storeArea: [],
      subBranch: [],
      defaultFileList: [],
      idCardFrontFileList: [],
      idCardBackFileList: [],
      storeInFileList: [],
      storeOutFileList: [],
      idCardFrontUploading: false,
      idCardBackUploading: false,
      storeInUploading: false,
      storeOutUploading: false,
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
  created () {
    this.getBankList()
    this.getRateList()
    this.getBankAreaList()
    this.getProductDescData()
  },
  methods: {
    // 身份证有效期选择
    idCardDateOnChange (date, dateString) {
      this.resultData.id_card_valid_time = dateString
    },
    getBankList () {
      this.$get('common/getParentBankList').then((r) => {
        let code = r.data.code
        if (code === 0) {
          this.bankData = r.data.data
        } else {
          console.warn(r.data.msg)
        }
      })
    },
    getRateList () {
      this.$get('common/getRateList').then((r) => {
        let code = r.data.code
        if (code === 0) {
          this.rateData = r.data.data
        } else {
          console.warn(r.data.msg)
        }
      })
    },
    getProductDescData () {
      this.productDescData = ['餐饮', '线下零售', '居民生活服务', '休闲娱乐', '交通出行', '其他']
    },
    getBankAreaList () {
      this.$get('common/getBankAreaList').then((r) => {
        let code = r.data.code
        if (code === 0) {
          let data = r.data.data
          this.bankArea = data
          this.storeArea = data
        } else {
          console.warn(r.data.msg)
        }
      })
    },
    // 银行ID选择
    handleBankChange (val) {
      this.bank.parentId = val
      this.resultData.bankId = val
    },
    // 银行 省市选择
    onChangeBankArea (val) {
      let length = val.length
      if (length === 3) {
        let bankAreaId = val[length - 1]
        this.resultData.bank_address_code = bankAreaId
      }
    },
    // 门店 省市选择
    onChangeStoreArea (val) {
      let length = val.length
      if (length === 3) {
        let storeAreaId = val[length - 1]
        this.resultData.store_address_code = storeAreaId
      }
    },
    // 加载银行 省市
    loadBankAreaData (selectedOptions) {
      const targetOption = selectedOptions[selectedOptions.length - 1]
      let parentId = targetOption.value
      let level = targetOption.level + 1
      let area = {
        'parentId': parentId,
        'level': level
      }
      // load options lazily
      setTimeout(() => {
        targetOption.loading = false
        this.$post('common/getChildrenArea', {
          ...area
        }).then((r) => {
          let code = r.data.code
          if (code === 0) {
            targetOption.children = r.data.data
            this.bankArea = [...this.bankArea]
          }
        }).catch(() => {
        })
      }, 1000)
    },
    // 加载门店 省市
    loadBankStoreData (selectedOptions) {
      const targetOption = selectedOptions[selectedOptions.length - 1]
      let parentId = targetOption.value
      let level = targetOption.level + 1
      let area = {
        'parentId': parentId,
        'level': level
      }
      // load options lazily
      setTimeout(() => {
        targetOption.loading = false
        this.$post('common/getChildrenArea', {
          ...area
        }).then((r) => {
          let code = r.data.code
          if (code === 0) {
            targetOption.children = r.data.data
            this.storeArea = [...this.storeArea]
          }
        }).catch(() => {
        })
      }, 1000)
    },
    // 费率选择
    handleRateChange (val) {
      this.resultData.rateId = val
    },
    // 商家描述
    handleProductDescChange (val) {
      this.resultData.product_desc = val
    },
    // 银行支行ID选择
    handleSubBankSelect (val) {
      this.resultData.subBankId = val
    },
    handleBranchSearch (val) {
      let parentId = this.bank.parentId
      let bank = {
        'parentId': parentId,
        'bankName': val
      }
      if (this.resultData.bankId == null || this.resultData.bankId === 0) {
        this.$message.warning('请先选择开户银行！再输入支行信息')
        return
      }
      if (val != null && val !== '') {
        this.$post('common/getBankListByName', {
          ...bank
        }).then((r) => {
          let code = r.data.code
          if (code === 0) {
            this.subBranch = r.data.data
          }
        }).catch(() => {
        })
      }
    },
    idCardFrontHandleRemove (file) {
      if (this.idCardFrontUploading) {
        this.$message.warning('文件导入中，请勿删除')
        return
      }
      const index = this.idCardFrontFileList.indexOf(file)
      const newFileList = this.idCardFrontFileList.slice()
      newFileList.splice(index, 1)
      this.idCardFrontFileList = newFileList
    },
    idCardBackHandleRemove (file) {
      if (this.idCardBackUploading) {
        this.$message.warning('文件导入中，请勿删除')
        return
      }
      const index = this.idCardBackFileList.indexOf(file)
      const newFileList = this.idCardBackFileList.slice()
      newFileList.splice(index, 1)
      this.idCardBackFileList = newFileList
    },
    storeInHandleRemove (file) {
      if (this.storeInUploading) {
        this.$message.warning('文件导入中，请勿删除')
        return
      }
      const index = this.storeInFileList.indexOf(file)
      const newFileList = this.storeInFileList.slice()
      newFileList.splice(index, 1)
      this.storeInFileList = newFileList
    },
    storeOutHandleRemove (file) {
      if (this.storeOutUploading) {
        this.$message.warning('文件导入中，请勿删除')
        return
      }
      const index = this.storeOutFileList.indexOf(file)
      const newFileList = this.storeOutFileList.slice()
      newFileList.splice(index, 1)
      this.storeOutFileList = newFileList
    },
    idCardFrontBeforeUpload (file) {
      this.idCardFrontFileList = [...this.idCardFrontFileList, file]
      return false
    },
    idCardBackBeforeUpload (file) {
      this.idCardBackFileList = [...this.idCardBackFileList, file]
      return false
    },
    storeInBeforeUpload (file) {
      this.storeInFileList = [...this.storeInFileList, file]
      return false
    },
    storeOutBeforeUpload (file) {
      this.storeOutFileList = [...this.storeOutFileList, file]
      return false
    },
    idCardFrontHandleUpload () {
      const { idCardFrontFileList } = this
      const formData = new FormData()
      formData.append('file', idCardFrontFileList[0])
      formData.append('fileType', '1')
      this.idCardFrontUploading = true
      this.$upload('/merchant/upload', formData).then((r) => {
        this.idCardFrontUploading = false
        this.idCardFrontFileList = []
        if (r.data.code === 0) {
          this.resultData.id_card_copy = r.data.data
          this.$message.success('上传成功')
        }
      }).catch((r) => {
        console.error(r)
        this.idCardFrontUploading = false
        this.idCardFrontFileList = []
        this.$message.error('上传失败')
      })
    },
    idCardBackHandleUpload () {
      const { idCardBackFileList } = this
      const formData = new FormData()
      formData.append('file', idCardBackFileList[0])
      formData.append('fileType', '2')
      this.idCardBackUploading = true
      this.$upload('/merchant/upload', formData).then((r) => {
        this.idCardBackUploading = false
        this.idCardBackFileList = []
        if (r.data.code === 0) {
          this.resultData.id_card_national = r.data.data
          this.$message.success('上传成功')
        }
      }).catch((r) => {
        console.error(r)
        this.idCardBackUploading = false
        this.idCardBackFileList = []
        this.$message.error('上传失败')
      })
    },
    storeInHandleUpload () {
      const { storeInFileList } = this
      const formData = new FormData()
      formData.append('file', storeInFileList[0])
      formData.append('fileType', '3')
      this.storeInUploading = true
      this.$upload('/merchant/upload', formData).then((r) => {
        this.storeInUploading = false
        this.storeInFileList = []
        if (r.data.code === 0) {
          this.resultData.store_entrance_pic = r.data.data
          this.$message.success('上传成功')
        }
      }).catch((r) => {
        console.error(r)
        this.storeInUploading = false
        this.storeInFileList = []
        this.$message.error('上传失败')
      })
    },
    storeOutHandleUpload () {
      const { storeOutFileList } = this
      const formData = new FormData()
      formData.append('file', storeOutFileList[0])
      formData.append('fileType', '4')
      this.storeOutUploading = true
      this.$upload('/merchant/upload', formData).then((r) => {
        this.storeOutUploading = false
        this.storeOutFileList = []
        if (r.data.code === 0) {
          this.resultData.indoor_pic = r.data.data
          this.$message.success('上传成功')
        }
      }).catch((r) => {
        console.error(r)
        this.storeOutUploading = false
        this.storeOutFileList = []
        this.$message.error('上传失败')
      })
    },
    onClose () {
      this.reset()
      this.$emit('close')
    },
    // 提交进件请求
    handleSubmit () {
      this.$post('merchant/merchantApply', {
        ...this.resultData
      }).then((r) => {
        let code = r.data.code
        if (code === 0) {
          this.$emit('success')
        }
      }).catch(() => {
      })
    }
  },
  watch: {
    userAddVisiable () {
    }
  }
}
</script>
<style lang="less" scoped>
  .option-area {
    display: inline-block;
    width: 100%;
    padding: 0 .9rem;
    margin: .5rem 0;
    .upload-area {
      display: inline;
      float: left;
      width: 50%
    }
    .button-area {
      margin-left: 1rem;
      display: inline;
      float: right;
      width: 30%;
    }
  }
</style>
