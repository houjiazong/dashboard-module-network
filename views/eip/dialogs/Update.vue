<template>
  <base-dialog @cancel="cancelDialog">
    <div slot="header">修改带宽</div>
    <div slot="body">
      <dialog-selected-tips :count="params.data.length" action="修改带宽" />
      <vxe-grid class="mb-2" :data="params.data" :columns="params.columns.slice(0, 3)" />
      <a-form
        :form="form.fc">
        <a-form-item label="带宽" v-bind="formItemLayout">
          <a-row>
            <a-col :span="12">
              <a-slider :min="1" :max="200" v-decorator="decorators.bandwidth" />
            </a-col>
            <a-col :span="4">
              <a-input-number :min="1" :max="200" v-decorator="decorators.bandwidth" />
            </a-col>
          </a-row>
        </a-form-item>
      </a-form>
    </div>
    <div slot="footer">
      <a-button type="primary" @click="handleConfirm" :loading="loading">{{ $t('dialog.ok') }}</a-button>
      <a-button @click="cancelDialog">{{ $t('dialog.cancel') }}</a-button>
    </div>
  </base-dialog>
</template>

<script>
import DialogMixin from '@/mixins/dialog'
import WindowsMixin from '@/mixins/windows'

export default {
  name: 'EipUpdateDialog',
  mixins: [DialogMixin, WindowsMixin],
  data () {
    return {
      loading: false,
      form: {
        fc: this.$form.createForm(this),
      },
      decorators: {
        bandwidth: [
          'bandwidth',
          {
            initialValue: 30,
          },
        ],
      },
      formItemLayout: {
        wrapperCol: {
          span: 21,
        },
        labelCol: {
          span: 3,
        },
      },
    }
  },
  methods: {
    doUpdate (data) {
      return this.params.list.singlePerformAction('change-bandwidth', data)
    },
    async handleConfirm () {
      this.loading = true
      try {
        const values = await this.form.fc.validateFields()
        await this.doUpdate(values)
        this.loading = false
        this.cancelDialog()
      } catch (error) {
        this.loading = false
      }
    },
  },
}
</script>
