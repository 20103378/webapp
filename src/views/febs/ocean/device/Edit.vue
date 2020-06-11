<template>
  <el-dialog
    :title="title"
    :width="width"
    top="50px"
    :close-on-click-modal="false"
    :close-on-press-escape="false"
    :visible.sync="isVisible"
  >
    <el-form ref="form" :model="device" :rules="rules" label-position="right" label-width="100px">
      <el-form-item :label="$t('table.device.deviceId')" prop="deviceId">
        <el-input v-model="device.deviceId" />
      </el-form-item>
      <el-form-item :label="$t('table.device.devicename')" prop="deviceName">
        <el-input v-model="device.deviceName"  />
      </el-form-item>
      <el-form-item :label="$t('table.device.latitude')" prop="latitude">
        <el-input v-model="device.latitude"  />
      </el-form-item>
      <el-form-item :label="$t('table.device.longitude')" prop="longitude">
        <el-input v-model="device.longitude"  />
      </el-form-item>
      <el-form-item :label="$t('table.device.topic')" prop="topic">
        <el-input v-model="device.topic"  />
      </el-form-item>
      <el-form-item :label="$t('table.device.samplePeriod')" prop="samplePeriod">
        <el-input v-model="device.samplePeriod"  />
      </el-form-item>

      <el-form-item :label="$t('table.status')" prop="status">
        <el-radio-group v-model="device.status">
          <el-radio label="1">{{ $t('common.status.valid') }}</el-radio>
          <el-radio label="0">{{ $t('common.status.invalid') }}</el-radio>
        </el-radio-group>
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button type="warning" plain :loading="buttonLoading" @click="isVisible = false">
        {{ $t('common.cancel') }}
      </el-button>
      <el-button type="primary" plain :loading="buttonLoading" @click="submitForm">
        {{ $t('common.confirm') }}
      </el-button>
    </div>
  </el-dialog>
</template>
<script>

export default {
  name: 'DeviceEdit',
  props: {
    dialogVisible: {
      type: Boolean,
      default: false
    },
    title: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      initFlag: false,
      device: this.initDevice(),
      buttonLoading: false,
      screenWidth: 0,
      width: this.initWidth(),
      rules: {
        deviceId: [
          { required: true, message: this.$t('rules.require'), trigger: 'blur' },
          { min: 1, max: 20, message: this.$t('rules.range1to20'), trigger: 'blur' }
        ],
        deviceName: [
          { required: true, message: this.$t('rules.require'), trigger: 'blur' },
          { min: 1, max: 20, message: this.$t('rules.range1to20'), trigger: 'blur' }
        ],
        latitude:[
          { required: true, message: this.$t('rules.require'), trigger: 'blur' },
          { type:'number', message: this.$t('rules.invalidNumber'), trigger: 'blur' }
        ],
        longitude:[
          { required: true, message: this.$t('rules.require'), trigger: 'blur' },
          { type:'number', message: this.$t('rules.invalidNumber'), trigger: 'blur' }
        ],
        topic: [
          { required: true, message: this.$t('rules.require'), trigger: 'blur' },
          { min: 1, max: 20, message: this.$t('rules.range1to20'), trigger: 'blur' }
        ],
        samplePeriod:[
          { required: true, message: this.$t('rules.require'), trigger: 'blur' },
          { type:'number', message: this.$t('rules.invalidNumber'), trigger: 'blur' }
        ],
        status: { required: true, message: this.$t('rules.require'), trigger: 'blur' }
      }
    }
  },
  computed: {
    isVisible: {
      get() {
        return this.dialogVisible
      },
      set() {
        this.close()
        this.reset()
      }
    }
  },
  mounted() {
    window.onresize = () => {
      return (() => {
        this.width = this.initWidth()
      })()
    }
  },
  methods: {
    initDevice() {
      return {
        id: '',
        deviceName: '',
        status: '1'
      }
    },
    initWidth() {
      this.screenWidth = document.body.clientWidth
      if (this.screenWidth < 991) {
        return '90%'
      } else if (this.screenWidth < 1400) {
        return '45%'
      } else {
        return '800px'
      }
    },
    setDevice(val) {
      this.device = { ...val }
    },
    close() {
      this.$emit('close')
    },
    submitForm() {
      this.$refs.form.validate((valid) => {
        if (valid) {
          this.buttonLoading = true
          if (!this.device.id) {
            // create
            this.$post('cable/device', { ...this.device }).then(() => {
              this.buttonLoading = false
              this.isVisible = false
              this.$message({
                message: this.$t('tips.createSuccess'),
                type: 'success'
              })
              this.$emit('success')
            })
          } else {
            // update
            this.device.createTime = this.device.updateTime = this.device.deleteTime = null
            this.$put('cable/device', { ...this.device }).then(() => {
              this.buttonLoading = false
              this.isVisible = false
              this.$message({
                message: this.$t('tips.updateSuccess'),
                type: 'success'
              })
              this.$emit('success')
            })
          }
        } else {
          return false
        }
      })
    },
    reset() {
      // 先清除校验，再清除表单，不然有奇怪的bug
      this.$refs.form.clearValidate()
      this.$refs.form.resetFields()
      this.device = this.initDevice()
    }
  }
}
</script>
<style lang="scss" scoped>
</style>
