<template>
  <a-card :body-style="{padding: '24px 32px'}" :bordered="false">
    <a-form
      style="margin: 0 auto;"
      @submit="handleSubmit"
      :form="form"
      :labelCol="{lg: {span: 7}, sm: {span: 7}}"
      :wrapperCol="{lg: {span: 10}, sm: {span: 17} }">

      <a-form-item label="名称">
        <a-input
          v-decorator="['env_name', {rules: [{required: true, message: '请输入名称'}, {max: 255, message: '名称超出长度限制'}]}]"
          placeholder="请输入名称"></a-input>
      </a-form-item>

      <a-form-item :wrapper-col="{ span: 12, offset: 7 }">
        <a-button :loading="saveLoading" htmlType="submit" type="primary">保存</a-button>
        <a-button style="margin-left: 16px" @click="cancelInfo">返回</a-button>
      </a-form-item>
    </a-form>
  </a-card>
</template>

<script>

import { postEnvironment, putEnvironment, getEnvironment } from '@/api/environment'

export default {
  name: 'ResourceAddOrEdit',
  // mixins: [apps],
  data () {
    return {
      form: this.$form.createForm(this),
      editId: this.$route.params.id || 'add',
      isAdd: true,
      pidOptions: [],
      data: [],
      saveLoading: false
    }
  },
  created () {
    // editId如果是add则认为是添加页面，否则认为是编辑页面
    if (this.editId === 'add') {
      this.isAdd = true
    } else {
      this.isAdd = false
      this.getItemDetail()
    }
  },
  methods: {
    handleChange (selectedItems) {
      console.log(selectedItems)
      this.selectedItems = selectedItems
    },
    /**
     * 编辑页需要获取当前编辑的资源详细信息，这里需要请
     * 求接口获取数据，并回填到表单里边
     */
    getItemDetail () {
      getEnvironment({ id: this.editId }).then(res => {
        if (res.code === 0) {
          const data = res.data
          this.data = data.members
          this.form.setFieldsValue({
            env_name: data.env_name
          })
        } else {
          this.$message.error(res.msg)
        }
      })
    },

    /**
     * 检查表单内容并提交动作
     * @param e
     */
    handleSubmit (e) {
      e.preventDefault()
      this.form.validateFieldsAndScroll((err, values) => {
        if (err) {
          return
        }

        const data = {
          ...values
        }
        this.saveLoading = true
        if (this.isAdd) {
          try {
            postEnvironment(data).then(res => {
              this.saveLoading = false
              if (res.code === 0) {
                this.$message.success(`添加成功`)
                this.$router.back()
              } else {
                this.$message.error(res.msg)
              }
            }).catch(() => {
              this.saveLoading = false
            })
          } catch (error) {
            console.log(error)
          }
        } else {
          data.status = 1
          putEnvironment({ ...data, id: ~~this.editId }).then(res => {
            this.saveLoading = false
            if (res.code === 0) {
              this.$message.success('更新成功')
              this.$router.back()
            } else {
              this.$message.error(res.msg)
            }
          }).catch(() => {
            this.saveLoading = false
          })
        }
      })
    },

    /**
     * 重置操作
     */
    handleReset () {
      this.form.resetFields(['name'])
    },

    /**
     * 取消按钮点击事件
     */
    cancelInfo () {
      this.$router.back()
    }
  }
}
</script>
