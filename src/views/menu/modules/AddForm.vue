<template>
  <a-modal
    title="新增菜单"
    :width="720"
    :visible="visible"
    :confirmLoading="confirmLoading"
    :maskClosable="false"
    @ok="handleSubmit"
    @cancel="handleCancel"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item label="菜单名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input
            v-decorator="['name', { rules: [{ required: true, min: 2, message: '请输入至少2个字符' }]} ]"
          />
        </a-form-item>
        <a-form-item label="上级菜单" :labelCol="labelCol" :wrapperCol="wrapperCol" extra="默认为顶级菜单">
          <a-tree-select
            :tree-data="menuListTreeData"
            :dropdown-style="{ maxHeight: '400px', overflow: 'auto' }"
            allow-clear
            v-decorator="['parent_id', { rules: [{ required: true }] }]"
          ></a-tree-select>
        </a-form-item>
        <a-form-item label="菜单path" :labelCol="labelCol" :wrapperCol="wrapperCol" extra="指向的页面path">
          <a-input v-decorator="['path', { rules: [{ required: true, message: '请输入菜单path' }] }]" />
        </a-form-item>
        <a-form-item
          label="是否为页面"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          extra="如果为否，则为纯菜单项，仅做用于父级菜单使用"
        >
          <a-radio-group
            v-decorator="['is_page', { initialValue: 1, rules: [{ required: true }] }]"
          >
            <a-radio :value="1">是</a-radio>
            <a-radio :value="0">否</a-radio>
          </a-radio-group>
        </a-form-item>
        <a-form-item label="功能模块key" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['module_key']" />
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
import * as Api from '@/api/store/menu'

export default {
  props: {
    menuList: {
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

      // 当前表单
      form: this.$form.createForm(this),

      // 菜单列表 树状结构
      menuListTreeData: []
    }
  },

  methods: {

    /**
     * 显示对话框
     */
    show () {
      // 显示窗口
      this.visible = true
      // 获取菜单列表
      this.getMenuList()
      // 设置默认值
      this.setFieldsValue()
    },

    /**
     * 设置默认值
     */
    setFieldsValue () {
      this.$nextTick(() => {
        this.form.resetFields()
        this.form.setFieldsValue({ parent_id: 0 })
      })
    },

    /**
     * 获取菜单列表
     */
    getMenuList () {
      const { menuList } = this
      // 格式化菜单列表
      const selectList = this.formatTreeData(menuList)
      // 顶级菜单
      selectList.unshift({
        title: '顶级菜单',
        key: 0,
        value: 0
      })
      this.menuListTreeData = selectList
    },

    /**
     * 格式化菜单列表
     */
    formatTreeData (list) {
      const data = []
      list.forEach(item => {
        const newItem = {
          title: item.name,
          key: item.menu_id,
          value: item.menu_id
        }
        if (item.hasOwnProperty('children')) {
          newItem['children'] = this.formatTreeData(item['children'])
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
      // 表单的验证
      validateFields((errors, values) => {
        if (!errors) {
          // 提交到后端api
          this.onFormSubmit(values)
        }
      })
    },

    /**
     * 取消按钮
     */
    handleCancel () {
      // 关闭窗口
      this.visible = false
      // 重置表单内容
      this.form.resetFields()
    },

    /**
    * 提交到后端api
    */
    onFormSubmit (values) {
      this.confirmLoading = true
      Api.add({ form: { type: 10, ...values } })
        .then((result) => {
          // 显示成功
          this.$message.success(result.message)
          // 关闭对话框
          this.handleCancel()
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
