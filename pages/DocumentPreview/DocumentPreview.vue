<template>
	<view class="uni-container">
		<page-head :title="title"></page-head>

		<view class="uni-padding-wrap uni-common-mt">
			<view class="uni-common-mt">
				<input class="uni-input" v-model="url" placeholder="请输入 http/https 文件地址（建议 pdf）" />
			</view>

			<view class="uni-btn-v uni-common-mt">
				<button type="default" @tap="testPing">测试插件 ping</button>
				<button type="primary" @tap="downloadAndOpen">下载并预览</button>
				<button type="primary" @tap="openUrlDirect">直接预览URL</button>
				<button type="primary" @tap="chooseAndOpen">选择文件并预览</button>
				<button type="primary" @tap="goEmbeddedPreview">组件内嵌预览</button>
				<button type="default" @tap="canPreviewCurrent">canPreview（当前文件）</button>
				<button type="default" @tap="isOpenedNow">isOpened</button>
				<button type="warn" @tap="closeNow">close</button>
			</view>

			<view class="uni-common-mt">
				<text>当前文件：</text>
				<text selectable>{{ currentPath || "(无)" }}</text>
			</view>

			<view class="uni-common-mt">
				<text>日志：</text>
				<view v-for="(l, i) in logs" :key="i">
					<text selectable>{{ l }}</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import {
		echo,
		ping,
		open,
		canPreview,
		isOpened,
		close,
		chooseFile
	} from '@/uni_modules/seal-office-online-uts'

	export default {
		data() {
			return {
				title: '文档预览插件测试',
				url: 'https://gitee.com/mirrors/pdf.js/raw/master/web/compressed.tracemonkey-pldi-09.pdf',
				currentPath: '',
				logs: []
			}
		},
		methods: {
			pushLog(msg) {
				const next = (this.logs || []).slice(-30)
				next.push(new Date().toISOString() + ' ' + msg)
				this.logs = next
			},
			notify(title) {
				this.pushLog('notify ' + (title || ''))
				console.log('notify', title)
			},
			goEmbeddedPreview() {
				uni.navigateTo({
					url: '/pages/DocumentPreview/EmbeddedPreview'
				})
			},
			testPing() {
				this.pushLog('tap testPing')
				try {
					const ret = ping('hello-seal-office')
					this.pushLog('ping return ' + JSON.stringify(ret))
					console.log('ping return', ret)
				} catch (e) {
					this.pushLog('ping throw ' + JSON.stringify(e))
					console.log('ping throw', JSON.stringify(e))
				}
			},
			downloadAndOpen() {
				this.pushLog('tap downloadAndOpen')
				if (!this.url) {
					this.notify('请输入URL')
					return
				}
				this.currentPath = this.url
				this.safeOpen({
					file: {
						url: this.url,
						fileName: this.guessFileName(this.url) || 'download.pdf'
					},
					success: () => {
						this.pushLog('downloadAndOpen success')
					},
					fail: (e) => {
						this.pushLog('downloadAndOpen fail ' + (e.errMsg || ''))
						this.notify(e.errMsg || '预览失败')
					}
				})
			},
			openUrlDirect() {
				this.pushLog('tap openUrlDirect → facade download first')
				this.downloadAndOpen()
			},
			guessFileName(p) {
				const q = (p || '').split('?')[0]
				const i = q.lastIndexOf('/')
				return i >= 0 ? q.substring(i + 1) : q
			},
			chooseAndOpen() {
				this.pushLog('tap chooseAndOpen')
				chooseFile({
					success: (res) => {
						this.pushLog('chooseFile success path=' + (res.path || ''))
						this.currentPath = res.path || ''
						this.openByPath(res.path || '', res.name || 'picked')
					},
					fail: (e) => {
						this.pushLog('chooseFile fail ' + (e.errMsg || ''))
						this.notify('选择失败:' + (e.errMsg || ''))
					}
				})
			},
			openByPath(path, fileName) {
				const that = this
				that.pushLog('openByPath path=' + JSON.stringify(path))
				console.log('openByPath', path, fileName)
				if (!path) {
					that.notify('无可预览文件')
					return
				}
				const file = path.indexOf('://') >= 0 ? { uri: path, fileName } : { path, fileName }
				console.log('file', file)
				that.safeOpen({
					file,
					success() {
						that.pushLog('openByPath success')
						console.log('preview opened')
					},
					onClose() {
						that.pushLog('onClose')
						that.notify('预览已关闭')
					},
					onReturn() {
						that.pushLog('onReturn')
						that.notify('已返回应用')
					},
					fail(e) {
						that.pushLog('openByPath fail ' + (e.errMsg || ''))
						that.notify(e.errMsg || '预览失败')
					}
				})
			},
			canPreviewCurrent() {
				this.pushLog('tap canPreviewCurrent')
				const that = this
				if (!that.currentPath) {
					that.notify('请先下载或选择文件')
					return
				}
				that.safeCanPreview({
					file: { path: that.currentPath },
					success(res) {
						that.pushLog('canPreview=' + res.canPreview)
						that.notify('canPreview=' + res.canPreview)
					},
					fail(e) {
						that.pushLog('canPreview fail ' + (e.errMsg || ''))
						that.notify(e.errMsg || 'canPreview失败')
					}
				})
			},
			isOpenedNow() {
				this.pushLog('tap isOpenedNow')
				const that = this
				that.safeIsOpened({
					success(res) {
						that.pushLog('opened=' + res.opened)
						that.notify('opened=' + res.opened)
					},
					fail(e) {
						that.pushLog('isOpened fail ' + (e.errMsg || ''))
						that.notify(e.errMsg || 'isOpened失败')
					}
				})
			},
			closeNow() {
				this.pushLog('tap closeNow')
				const that = this
				that.safeClose({
					success() {
						that.pushLog('close success')
						that.notify('已关闭')
					},
					fail(e) {
						that.pushLog('close fail ' + (e.errMsg || ''))
						that.notify(e.errMsg || 'close失败')
					}
				})
			},
			safeOpen(options) {
				this.pushLog('call open')
				try {
					console.log('safeOpen options', options ? JSON.stringify(options) : 'null')
				} catch (e) {
					console.log('safeOpen throw ' + JSON.stringify(e))
				}
				try {
					const file = options ? options.file : null
					this.pushLog('open options file=' + JSON.stringify(file || null))
					open(options)
					this.pushLog('open called')
				} catch (e) {
					console.log('openImpl throw ' + JSON.stringify(e))
					this.notify('open异常')
				}
			},
			safeCanPreview(options) {
				this.pushLog('call canPreview')
				try {
					canPreview(options)
					this.pushLog('canPreview called')
				} catch (e) {
					this.pushLog('canPreview throw ' + JSON.stringify(e))
					this.notify('canPreview异常')
				}
			},
			safeIsOpened(options) {
				this.pushLog('call isOpened')
				try {
					isOpened(options)
					this.pushLog('isOpened called')
				} catch (e) {
					this.pushLog('isOpened throw ' + JSON.stringify(e))
					this.notify('isOpened异常')
				}
			},
			safeClose(options) {
				this.pushLog('call close')
				try {
					close(options)
					this.pushLog('close called')
				} catch (e) {
					this.pushLog('close throw ' + JSON.stringify(e))
					this.notify('close异常')
				}
			}
		}
	}
</script>

<style>
	@import '@/common/uni-uvue.css';

	.uni-container {
		min-height: 100%;
	}
</style>
