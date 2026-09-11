<template>
  <view class="page">
    <page-head title="文档预览组件测试"></page-head>
    <view class="page__body">
      <input class="page__input" v-model="url" placeholder="请输入 http/https 文件地址" />
      <view class="page__row">
        <input class="page__input" v-model="fileName" placeholder="fileName（例：demo.pdf）" />
        <input class="page__input" v-model="fileType" placeholder="fileType（pdf/docx/xlsx/pptx）" />
      </view>
      <view class="page__actions">
        <button type="primary" @tap="applyUrl">设置组件 source</button>
        <button type="primary" @tap="openByRef">调用组件 openFullscreen</button>
        <button type="default" @tap="reloadByRef">调用组件 reload</button>
      </view>
      <view class="page__preview">
        <seal-office-online-uts
          ref="preview"
          :source="source"
          :autoLoad="true"
          :autoOpen="false"
          @ready="pushLog('ready')"
          @load="handleLoad"
          @error="handleError"
          @requestfullscreen="pushLog('requestfullscreen')"
          @fullscreenopened="pushLog('fullscreenopened')"
          @tap="pushLog('tap')"
        />
      </view>
      <view class="page__logs">
        <text>当前状态：{{ stateText }}</text>
        <view v-for="(log, index) in logs" :key="index">
          <text selectable>{{ log }}</text>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      url: 'https://www.w3.org/WAI/ER/tests/xhtml/testfiles/resources/pdf/dummy.pdf',
      fileName: 'dummy.pdf',
      fileType: 'pdf',
      source: {},
      logs: [],
      stateText: 'idle'
    }
  },
  methods: {
    pushLog(message) {
      const next = (this.logs || []).slice(-20)
      next.push(new Date().toISOString() + ' ' + message)
      this.logs = next
    },
    applyUrl() {
      this.source = {
        url: this.url,
        fileName: this.fileName,
        title: '组件预览文档',
        fileType: this.fileType
      }
      this.pushLog('applyUrl')
    },
    openByRef() {
      const p = this.$refs.preview
      if (!p) return
      p.openFullscreen()
      this.stateText = p.getState()
      this.pushLog('openByRef')
    },
    reloadByRef() {
      const p = this.$refs.preview
      if (!p) return
      p.reload()
      this.stateText = p.getState()
      this.pushLog('reloadByRef')
    },
    handleLoad() {
      const p = this.$refs.preview
      this.stateText = p ? p.getState() : this.stateText
      this.pushLog('load state=' + this.stateText)
    },
    handleError(err) {
      const p = this.$refs.preview
      this.stateText = p ? p.getState() : this.stateText
      this.pushLog('error ' + ((err && err.errMsg) || 'unknown'))
    }
  },
  onLoad() {
    this.applyUrl()
  }
}
</script>

<style>
.page {
  min-height: 100%;
}

.page__body {
  padding: 24rpx;
}

.page__input {
  border-width: 1px;
  border-style: solid;
  border-color: #d9d9d9;
  padding: 16rpx;
  margin-bottom: 24rpx;
}

.page__row {
  display: flex;
  flex-direction: row;
  column-gap: 16rpx;
}

.page__actions {
  margin-bottom: 24rpx;
}

.page__preview {
  min-height: 420rpx;
  background-color: #f5f5f5;
  margin-bottom: 24rpx;
}

.page__logs {
  background-color: #ffffff;
  padding: 16rpx;
}
</style>
