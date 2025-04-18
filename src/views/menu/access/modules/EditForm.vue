<template>
  <a-modal
    title="编辑权限"
    :width="720"
    :visible="visible"
    :confirmLoading="confirmLoading"
    :maskClosable="false"
    @ok="handleSubmit"
    @cancel="handleCancel"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item label="权限名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input
            v-decorator="['name', {rules: [{required: true, min: 2, message: '请输入至少2个字符'}]}]"
          />
        </a-form-item>
        <a-form-item label="上级权限" :labelCol="labelCol" :wrapperCol="wrapperCol" extra="默认为顶级权限">
          <a-tree-select
            :tree-data="apiListTreeData"
            :dropdown-style="{ maxHeight: '400px', overflow: 'auto' }"
            allow-clear
            v-decorator="['parent_id']"
          ></a-tree-select>
        </a-form-item>
        <a-form-item
          label="权限url"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          extra="例如：index/index"
        >
          <a-input v-decorator="['url', {rules: [{required: true, message: '请输入权限url'}]}]" />
        </a-form-item>
        <a-form-item label="排序" :labelCol="labelCol" :wrapperCol="wrapperCol" extra="数字越小越靠前">
          <a-input-number
            :min="0"
            v-decorator="['sort', {initialValue: 100, rules: [{required: true, message: '请输入至少1个数字'}]}]"
          />
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
import pick from 'lodash.pick'
import * as Api from '@/api/store/api'

export default {
  props: {
    apiList: {
      type: Array,
      required: true
    }
  },
  data () {
    return {
      // 标签布局属性
      labelCol: {
        xs: { span: 24 },
        sm: { span: 7 }
      },
      // 输入框布局属性
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 13 }
      },
      // modal(对话框)是否可见
      visible: false,
      // modal(对话框)确定按钮 loading
      confirmLoading: false,

      form: this.$form.createForm(this),

      // 权限列表 树状结构
      apiListTreeData: [],

      // 当前记录
      record: {}

    }
  },

  methods: {

    /**
     * 显示对话框
     */
    show (item) {
      // 显示窗口
      this.visible = true
      this.record = item
      // 获取权限列表
      this.getAccessList()
      // 设置默认值
      this.setFieldsValue()
    },

    /**
     * 设置默认值
     */
    setFieldsValue () {
      this.$nextTick(() => {
        this.form.setFieldsValue(pick(this.record, ['name', 'url', 'parent_id', 'sort']))
      })
    },

    /**
     * 获取权限列表
     */
    getAccessList () {
      const { apiList } = this
      // 格式化权限列表
      const selectList = this.formatTreeData(apiList)
      // 顶级权限
      selectList.unshift({
        title: '顶级权限',
        key: 0,
        value: 0
      })
      this.apiListTreeData = selectList
    },

    /**
     * 格式化权限列表
     */
    formatTreeData (list, disabled = false) {
      const data = []
      list.forEach(item => {
        // 新的元素
        const newItem = {
          title: item.name,
          key: item.api_id,
          value: item.api_id
        }
        // 禁用的权限
        if (
          [item.api_id, item.parent_id].includes(this.record.api_id) ||
          disabled === true
        ) {
          newItem.disabled = true
        }
        // 递归整理子集
        if (item.children != null) {
          newItem['children'] = this.formatTreeData(item['children'], newItem.disabled)
        }
        data.push(newItem)
      })
      return data
    },

    /**
     * 确认按钮
     */
    handleSubmit () {
      const { form: { validateFields } } = this
      this.confirmLoading = true
      // 表单的验证
      validateFields((errors, values) => {
        if (!errors) {
          // 提交到后端api
          this.onFormSubmit(values)
        } else {
          this.confirmLoading = false
        }
      })
    },

    /**
     * 取消按钮
     */
    handleCancel () {
      this.visible = false
    },

    /**
    * 提交到后端api
    */
    onFormSubmit (values) {
      Api.edit({ apiId: this.record['api_id'], form: values })
        .then((result) => {
          // 显示成功
          this.$message.success(result.message)
          // 关闭窗口
          this.visible = false
          // 重置表单内容
          this.form.resetFields()
          // 通知父端组件提交完成了
          this.$emit('handleSubmit', values)
        })
        .finally((result) => {
          this.confirmLoading = false
        })
    }

  }
}
</script>
