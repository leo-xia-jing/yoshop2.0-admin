<template>
  <a-card :bordered="false">
    <content-header title="商城列表"></content-header>
    <div class="table-operator">
      <a-button type="primary" icon="plus" @click="$refs.createModal.show()">新增</a-button>
    </div>
    <s-table
      ref="table"
      size="default"
      rowKey="store_id"
      :columns="columns"
      :data="loadData"
      showPagination="auto"
      :pageSize="15"
    >
      <span class="actions" slot="action" slot-scope="text, item">
        <a @click="handleInto(item)">进入商城</a>
        <a @click="handleDelete(item)">删除</a>
        <!-- <a @click="handleModule(item)">功能模块</a> -->
      </span>
    </s-table>
    <create-form ref="createModal" @handleSubmit="handleRefresh" />
  </a-card>
</template>

<script>
import * as Api from '@/api/store'
import { ContentHeader, STable } from '@/components'
import CreateForm from './modules/CreateForm'
import { urlEncode } from '@/utils/util'

export default {
  components: {
    ContentHeader,
    STable,
    CreateForm
  },
  data () {
    return {
      // 查询参数
      queryParam: {},
      // 表头
      columns: [
        {
          title: '商城ID',
          dataIndex: 'store_id'
        },
        {
          title: '商城名称',
          dataIndex: 'store_name'
        },
        {
          title: '排序',
          dataIndex: 'sort'
        },
        {
          title: '添加时间',
          dataIndex: 'create_time'
        },
        {
          title: '操作',
          dataIndex: 'action',
          width: '200px',
          scopedSlots: { customRender: 'action' }
        }
      ],
      // 加载数据方法 必须为 Promise 对象
      loadData: param => {
        return Api.list(Object.assign(param, this.queryParam))
          .then(response => {
            return response.data.list
          })
      }
    }
  },
  created () {
  },
  methods: {

    /**
     * 进入商城
     */
    handleInto (item) {
      if (process.env.NODE_ENV !== 'production') {
        this.$message.error('很抱歉，开发环境中不支持')
        return false
      }
      const storeUrl = window.serverConfig.STORE_URL
      const url = `${storeUrl}/#/passport/login`
      window.open(url, '_blank')
    },

    /**
     * 删除记录
     */
    handleDelete (item) {
      const app = this
      app.$confirm({
        title: '您确定要删除该商城吗?',
        content: '确认后将移入回收站',
        // okType: 'danger',
        onOk () {
          // 确认删除
          return app.onSubmitDelete(item)
        }
      })
    },

    /**
     * 确认删除
     */
    onSubmitDelete (item) {
      return Api.recovery({ storeId: item['store_id'] })
        .then(result => {
          this.$message.success(result.message)
          this.handleRefresh()
        })
    },

    // // 分配功能模块
    // handleModule (item) {
    //   // 显示对话框
    //   this.$refs.ModuleForm.show(item)
    // },

    /**
     * 刷新列表
     */
    handleRefresh () {
      this.$refs.table.refresh()
    }

  }
}
</script>
