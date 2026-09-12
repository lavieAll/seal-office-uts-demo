# seal-office-online-uts 文档预览插件

Demo：[Gitee](https://gitee.com/twofloor/seal-office-uts-demo) · [GitHub](https://github.com/lavieAll/seal-office-uts-demo)

`seal-office-online-uts` 是面向 uni-app、uni-app x 的 UTS 文档预览插件，提供统一函数式 API 和可嵌入页面的 `<seal-office-online-uts>` 组件，可预览 PDF、Word、Excel、PowerPoint 文件。

👏👏👏欢迎加WX（BJGFCYY）或Q（2480621579）咨询。

## 支持平台

| 平台 | 实现方式 |
| --- | --- |
| App Android | 内置 Seal Reader，支持本地路径、URI 和网络地址 |
| App iOS | Quick Look 预览 |
| 鸿蒙 Next App | PreviewKit；不可用时回退系统文件查看器 |
| 微信、支付宝、百度、QQ、快手、飞书、京东、小红书、360、抖音小程序 | 调用平台 `openDocument`，网络文件先下载 |
| 华为快应用、联盟快应用 | 调用快应用文档查看能力，网络文件先缓存 |
| H5/Web | API 保持一致，但浏览器不提供原生文档查看，打开接口返回“不支持” |

插件版本 4.0.1，要求 HBuilderX `^4.61`、uni-app/uni-app x `^3.8.3`。插件不采集数据，也不声明额外权限。

## 安装

将 `uni_modules/seal-office-online-uts` 放入项目 `uni_modules` 目录（或从插件市场导入），重新打开 HBuilderX 后运行或发行目标平台。请使用真机或对应平台开发者工具验证原生能力。

## 文件描述（FileSource）

```ts
type FileSource = {
  path?: string       // 本地绝对路径或临时文件路径
  uri?: string        // content://、file:// 等 URI
  url?: string        // http/https 地址
  fileName?: string   // 文件名，建议带扩展名
  fileType?: string   // pdf/doc/docx/xls/xlsx/ppt/pptx，可带或不带点
  title?: string      // 展示标题
  mimeType?: string   // MIME 类型（可选）
}
```

至少提供 `path`、`uri` 或 `url` 之一。`fileType` 未填写时会从文件名或路径扩展名推断。网络地址必须可从设备或小程序运行环境访问；需要鉴权时请先下载文件，再传入本地路径。

## 函数式 API

```ts
import { open, openFiles, canPreview, isOpened, close, chooseFile, ping, echo }
  from '@/uni_modules/seal-office-online-uts'
```

### 打开文件

```ts
open({
  file: { url: 'https://example.com/report.pdf', fileName: 'report.pdf', title: '月度报告' },
  navBar: { title: '文档预览', showMore: true, backgroundColor: '#1677ff', textColor: '#fff', textSize: 16 },
  success(res) { console.log('打开成功', res) },
  fail(err) { console.error(err.errCode, err.errMsg) },
  complete(res) {},
  onClose() { console.log('预览关闭') },
  onReturn() { console.log('返回应用') }
})
```

支持 `path`、`uri`、`url` 三种来源。网络文件在支持的平台上会自动下载。`navBar` 可配置 `title`、`showEdit`、`showMore`、`backgroundColor`、`textColor`、`textSize`。

### 检查、多文件、状态和关闭

```ts
canPreview({ file: { path: '/path/report.pdf' }, success(res) { console.log(res.canPreview) } })
openFiles({ files: [
  { path: '/path/a.pdf', fileName: 'a.pdf' },
  { path: '/path/b.docx', fileName: 'b.docx' }
], index: 0, success(res) {}, fail(err) {} })
isOpened({ success(res) { console.log(res.opened) } })
close({ success(res) {}, fail(err) {} })
```

小程序和快应用的 `isOpened` 通常返回 `opened: false`，`close` 是否可用由平台决定，请以回调结果为准。

### 选择文件与诊断

```ts
chooseFile({
  success(res) { open({ file: { path: res.path, fileName: res.name } }) },
  fail(err) {}
})
const value = ping('hello') // platform:hello
echo({ value: 'hello', success(res) { console.log(res.value) } })
```

`chooseFile` 依赖当前平台的 `uni.chooseFile`，不支持时触发 `fail`。

## 内嵌预览组件

示例见 `pages/DocumentPreview/EmbeddedPreview.vue` 和 `.uvue`。

```vue
<template>
  <seal-office-online-uts ref="preview" :source="source" :autoLoad="true" :autoOpen="false"
    :showToolbar="true" backgroundColor="#f5f5f5"
    @ready="onReady" @load="onLoad" @error="onError"
    @requestfullscreen="onRequest" @fullscreenopened="onOpened" @close="onClose" @tap="onTap" />
</template>
<script>
export default {
  data() { return { source: { url: 'https://example.com/report.pdf', fileName: 'report.pdf', fileType: 'pdf', title: '报告' } } },
  methods: { onReady(e) {}, onLoad(e) {}, onError(e) {}, onRequest(e) {}, onOpened(e) {}, onClose(e) {}, onTap(e) {} }
}
</script>
```

组件属性：`source`（默认 `{}`）、`autoLoad`（`true`）、`autoOpen`（`false`）、`fallbackToFullscreen`（`true`）、`backgroundColor`（`#F5F5F5`）、`emptyText`（`请选择文件`）、`showToolbar`（`true`）、`fitMode`（`contain`）。

通过 `ref` 调用实例方法：

```js
const preview = this.$refs.preview
preview.setSource({ path: '/path/report.pdf', fileName: 'report.pdf' })
preview.reload()
preview.openFullscreen()
console.log(preview.getState()) // idle/loading/ready/error
preview.dispose()
```

事件：`ready`、`load`、`error`、`tap`、`requestfullscreen`、`fullscreenopened`、`close`、`pagechange`。参数可能包含 `state`、`source`、`errCode`、`errMsg`。

## 平台注意事项

- 支持类型：`pdf`、`doc`、`docx`、`xls`、`xlsx`、`ppt`、`pptx`；扩展名缺失或不支持返回 `1103`。
- 小程序/快应用网络预览依赖合法域名、下载权限及平台的 `openDocument` 能力。
- 抖音小程序受系统策略影响：Android 不支持，iOS 的 `ppt` 可能不支持，建议先调用 `canPreview`。
- H5 仅提供接口占位实现，不能直接在浏览器内显示 Office 文档。
- iOS 使用 Quick Look，Android 使用 Seal Reader；无可用查看器时请在 `fail` 中提示用户。

## 错误处理

失败回调通常包含 `errSubject: 'uni-documentPreview'`、`errCode`、`errMsg`：

| 错误码 | 含义 |
| --- | --- |
| 1101 | 参数无效 |
| 1102 | 文件不存在或无法读取 |
| 1103 | 文件类型不支持 |
| 1104 | 当前平台或能力不支持 |
| 1105 | 无法取得原生上下文 |
| 1106 | 网络文件下载失败 |

建议始终提供 `success`、`fail`、`complete`，并在打开前检查 URL、扩展名和鉴权状态。

## 示例入口

- `pages/DocumentPreview/DocumentPreview.vue`：下载、URL 直开、选择文件、状态查询和关闭。
- `pages/DocumentPreview/DocumentPreview.uvue`：uni-app x 写法及跨端条件编译。
- `pages/DocumentPreview/EmbeddedPreview.vue`、`EmbeddedPreview.uvue`：组件 `source`、事件与实例方法。
