<template>
  <el-form :data="app" label-width="80px" label-position="left">
    <el-form-item label="app" :rules="[{required: true}]" v-if="isAdd">
      <el-upload drag action="/" :on-change="onChange" :multiple="false" :auto-upload="false">
        <i class="el-icon-upload" /><div>将app拖到此处，或点击选择app</div>
      </el-upload>
    </el-form-item>
    <el-form-item label="app名" :rules="[{required: true}]">
      <el-input v-model.trim="app.name" clearable style="width: 300px" />
    </el-form-item>
    <el-form-item label="version">
      <el-input v-model.trim="app.version" clearable style="width: 300px" />
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="saveApp" :loading="saveBtnLoading">保 存</el-button>
    </el-form-item>
  </el-form>
</template>
<script>

import { uploadApp, updateApp, getAppList } from '@/api/app'

export default {
  props: {
    isAdd: Boolean
  },
  data() {
    return {
      choosedApp: null,
      saveBtnLoading: false,
      app: {
        id: undefined,
        name: '',
        version: '',
        platform: this.$store.state.project.platform,
        projectId: this.$store.state.project.id
      }
    }
  },
  created() {
    if (!this.isAdd) {
      getAppList({ id: this.$route.params.appId }).then(response => {
        this.app = response.data[0]
      })
    }
  },
  methods: {
    // 选择app
    onChange(file) {
      this.choosedApp = file
    },
    saveAppSuccess(msg) {
      this.$notify.success(msg)
      // 关闭当前tagview
      this.$store.dispatch('tagsView/delView', this.$store.state.tagsView.visitedViews.filter(item => item.path === this.$route.path)[0])
      this.$router.back()
    },
    saveApp() {
      this.saveBtnLoading = true
      if (this.isAdd) {
        if (this.choosedApp == null) {
          this.$notify.error('请选择一个app')
          this.saveBtnLoading = false
          return
        }
        const formData = new FormData()
        formData.append('file', this.choosedApp.raw)
        for (const k in this.app) {
          if (this.app[k]) {
            formData.append(k, this.app[k])
          }
        }
        uploadApp(formData).then(response => {
          this.saveAppSuccess(response.msg)
        }).finally(() => {
          this.saveBtnLoading = false
        })
      } else {
        updateApp(this.app).then(response => {
          this.saveAppSuccess(response.msg)
        }).finally(() => {
          this.saveBtnLoading = false
        })
      }
    }
  }
}
</script>
