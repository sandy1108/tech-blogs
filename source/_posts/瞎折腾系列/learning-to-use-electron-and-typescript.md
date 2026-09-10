---
title: Electron与TypeScript的折腾
categories: 
  - 瞎折腾系列
excerpt: 本文主要是在空闲时间写点文章的时候，突然觉得编写hexo文章的步骤稍微有点繁琐，虽然也有一些Web的控制台插件，不过并没有去挨个尝试，因为本地操作其实就是一些目录目录，文件头，图片的存储，而很久之前就想要接触一下ElectronJS来弄一个Windows程序试试，这次干脆咬咬牙把这个想法实现了吧。（Flutter发展也很迅速，不过还是想先把之前的ElectronJS的“想法”实现了吧。）
date: 2024-08-26 16:50:32
tags:
  - Electron
  - TypeScript
  - Vue3
  - 桌面应用
  - IPC
  - Markdown编辑器
---

# ElectronJS与Typescript的折腾

## 写在前面

本文主要是在空闲时间写点文章的时候，突然觉得编写hexo文章的步骤稍微有点繁琐，虽然也有一些Web的控制台插件，不过并没有去挨个尝试，因为本地操作其实就是一些目录目录，文件头，图片的存储，而很久之前就想要接触一下ElectronJS来弄一个Windows程序试试，这次干脆咬咬牙把这个想法实现了吧。（Flutter发展也很迅速，不过还是想先把之前的ElectronJS的“想法”实现了吧。）

## 选型

由于是自己做着玩的工具软件，所以我的选型主旨要考虑两点：一是技术不能太陈旧，应该学习新技术；二是不选太不稳定的方案，没有太大意义。所以，在技术架构方面，本来其实很想使用自己较为熟悉的Vue2，搭配Vant的UI框架，然后结合到Electron官方的工程结构中。但是想来Vue3已经推出好久了，Vue2也停更了，TypeScript作为成熟又陌生的语言，也是时候上手搞一下了。（在搞之前，心里想的是，反正最近也搞了一阵子鸿蒙了，ArkTS不就是TypeScript的超集么，应该也差不多吧。不过后来发现还是略有区别。）

最后经过对比，我选择了Electron-Vite + Vue3 + NaiveUI这样的组合方式，不能说是最佳选择，只能说是方案不差，且满足了我想要尝试和学习新东西的需求。

具体版本记录如下：

```
"electron": "^31.0.2",
"electron-builder": "^24.13.3",
"electron-vite": "^2.3.0",
"vite": "^5.3.1",
"vue": "^3.4.30",
"naive-ui": "^2.39.0",
```

## 开始使用electron-vite + Vue3

直接参考了文档说明，新建工程。

https://cn.electron-vite.org/guide/

```
npm create @quick-start/electron my-app -- --template vue
```

然后

```
npm i
```

最后

```
npm run dev
```

嗯，可以，这就成了。

不过这其中遇到了些许问题，例如，我本地的node版本太旧了，人家要求Node18+，我才14。又升了一波Node版本，顺便配置了fnm作为Node版本管理工具，具体可以看看我另一篇记录文章。

## Electron基础概念

- Main Process 主进程

只有一个，用于与操作系统交互和全局管理等

- Renderer Process 渲染进程

每打开一个主BrowserWindow窗口就会有一个RendererProcess

- IPC 进程间通讯

主进程和渲染进程之间交互，一般来说，渲染进程中执行Web页面的代码，需要执行Node代码与操作系统API交互的，就需要通过IPC发送消息给主进程，在主进程监听对应的消息，执行操作，然后将操作结果返回给渲染进程。

如果做过移动端Hybrid开发的话就会发现，其实思路是一样的。虽然Electron中都是JS或者TS写，但是默认情况下，渲染进程中，也就是Web页面中是不能直接使用Node的很多API来与操作系统交互的。

- 那么怎么区分不同的进程呢？怎么知道哪些代码运行在哪个进程？

使用electron-vite这个框架创建的工程中，src目录中有main目录、preload目录、renderer目录，main/index.ts以及其引用的文件都会执行在主进程中，而renderer/index.ts及其引用的文件都会执行在渲染进程中，例如初始化出来的Vue框架。

## 使用IPC交互来让UI调用Node能力的一般做法

1. 首先可以在主进程中注册一个交互事件（应返回Promise，或者使用async方法的语法糖直接return）：

```
  ipcMain.handle('getFileListByPath', async (_event, dirPath: string) => {
    console.log('ipcMain.handle: getFileListByPath', dirPath)
    return ApiFile.getFileListByPath(dirPath)
  })
```

2. 然后在preload.js中，将接口注入在Window对象中：

```
// Custom APIs for renderer
const api = {
  getFileListByPath: async (dirPath: string) => {
    return electronAPI.ipcRenderer.invoke('getFileListByPath', dirPath)
  },
} as API
// Use `contextBridge` APIs to expose Electron APIs to
// renderer only if context isolation is enabled, otherwise
// just add to the DOM global.
if (process.contextIsolated) {
  try {
    contextBridge.exposeInMainWorld('electron', electronAPI)
    contextBridge.exposeInMainWorld('api', api)
  } catch (error) {
    console.error(error)
  }
} else {
  // @ts-ignore (define in dts)
  window.electron = electronAPI
  // @ts-ignore (define in dts)
  window.api = api
```

3. 最后在renderer进程中调用即可:

```
window.api.getFileListByPath(blogDirectoryHome).then((resultStr: string) => {
    const result: FileEntity[] = JSON.parse(resultStr)
    this.categoriesDirectoryList = result
})
```

## Markdown编辑组件 - v-md-editor

### v-md-editor的基本使用

https://code-farmer-i.github.io/vue-markdown-editor/zh/

使用的版本是"@kangc/v-md-editor": "^2.3.18"

我采用了其中的进阶版编辑器，包含了CodeMirror库。

但是在实际使用过程中，编译总是找不到CodeMirror，但是看了一下v-md-editor中已经有了依赖。但是由于新版本的v-md-editor需要自行导入CodeMirror，因此就需要自行import，便遇到了找不到CodeMirror的问题。

那简单，咱就直接npm i codemirror不就行了？不好意思，又报错了。可能是因为CodeMirror已经更新了，与v-md-editor不适配了。

于是去node_modules里找到v-md-editor，看到其中依赖的CodeMirror版本是5.65.17。

OK，安装，终于可用。

```
npm i codemirror@5.65.17
```

### 上传图片操作

我是要利用它的上传图片操作，实际上是把图片放到相对md文件的同名文件夹内，以便hexo中可以显示。于是我们依然监听一下上传图片的回调，然后在发送消息交给Electron主进程，将对应图片存放到对应位置即可：

```
    handleUploadImage(event, insertImage, files) {
      if (files.length === 0) return
      const filePaths: string[] = []
      files.forEach((file) => {
        filePaths.push(file.path)
      })
      // 把md文件的文件名作为同级目录
      const targetDirPath = this.filePath?.split('.md')[0]
      window.api.uploadImagesToLocalDirectory(targetDirPath, filePaths).then((res) => {
        console.log('uploadImagesToLocalDirectory: ', res)
      })
      console.log('handleUploadImage: ', event, insertImage, files)
      insertImage({
        url: files[0].name,
        desc: files[0].name
      })
      this.messageHandler.success('图片插入成功', {
        duration: 1500
      })
    },
```

## 多窗口的处理

创建模板工程后在main/index.ts中已经有了创建主窗口的代码，这是我们Electron程序启动时候的首页窗口。一般使用Vue开发的各种交互逻辑现在也大多是单页面的，内部页面切换可以使用router、tab等形式。

但是有的情况下还是想开启一个独立窗口进行操作。例如我想要为每一篇文章单独开启一个Markdown编辑器窗口。我的做法是，稍稍改造一下创建主窗口的方法，让它可以复用，传递用于区分窗口功能的参数。然后在Vue页面中接收参数后，判断显示的内容即可。

打开窗口并传递参数的办法，我采用发送消息的方式：

```
  mainWindow.webContents.once('did-finish-load', () => {
    WLog.log('mainWindow.webContents did-finish-load')
    if (paramStr) {
      mainWindow.webContents.send('initParams', paramStr)
    }
    mainWindow.show()
  })
```

然后在App.vue中接收参数：

```
const paramsObj: PageParam = reactive({
  pageName: '',
  paramStr: ''
} as PageParam)

window.electron.ipcRenderer.on('initParams', (_event, inParams: string) => {
  console.log('initParams: ', inParams)
  const inParamsObj = JSON.parse(inParams) as PageParam
  paramsObj.pageName = inParamsObj.pageName
  paramsObj.paramStr = inParamsObj.paramStr
})
```

并进行简单的判断：

```
<div v-if="paramsObj.pageName === 'main'" class="app-container-wrapper">
    <n-tabs class="tabs-frame" type="card" placement="left" animated>
      <n-tab-pane name="main">
        <template #tab> 文章列表 </template>
        <template #default>
          <BlogList />
        </template>
      </n-tab-pane>
    </n-tabs>
</div>
<div v-if="paramsObj.pageName === 'editor'" class="app-container-wrapper">
    <MarkdownEditor :file-path="paramsObj.paramStr" />
</div>
```

## 窗口标题栏以及滚动条美化

参考了这个文章的实践方案，https://zhuanlan.zhihu.com/p/677492706

主要思路是，new BrowserWindow的时候，禁用窗口自带的顶部标题栏，只保留最小化，最大化，关闭这种操作按钮。

```
  const mainWindow = new BrowserWindow({
    ...
    ...
    titleBarStyle: 'hidden',
    titleBarOverlay: {
      color: '#101014',
      symbolColor: '#74b1be',
      height: 35
    },
    ...
    ...
```

然后页面顶部预留一个横条div作为自定义标题栏，高度与按钮设置的一致，这样整体比较协调。

最后给自定义div设置可以拖拽。下面是示例代码：


```
/* 窗口标题栏占位 */
.app-title-bar {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  justify-content: center;
  height: 35px;
  width: 100%;
  background-color: #101014;
  /* 设置该属性表明这是可拖拽区域，用来移动窗口 */
  -webkit-app-region: drag;
}
```

## 因“懵懂”而遇到的一些问题

### 问题1：事件回调方法的第一个参数event我用不上，但是总是警告，编译报错

解决办法：参数名```event```改为```_event```

### 问题2：TypeScript的类型检查和声明

由于以前搞Android Java开发，总觉得JSON解析后需要有个class。在鸿蒙ArkTS开发的时候我会定义一些class。但是似乎TypeScript的很多写法里都只是定义一个type或者interface即可。这次既然是使用纯TypeScript，还是要按照真正的套路去写比较好。大概学习了一下他们的区别是，class在编译时不会剔除，它可以包含一些实例化时候的内部逻辑，而type和interface主要用来编译时进行检查，编译产物中会被移除。type还可以组合多种类型，也就是那种联合类型。

所以，以我目前的理解，用来处理JSON解析的实体类型的比较合适的方案应该是：使用interface定义一个JSON对象中会有哪些属性及其类型。class和type则在其他场景按实际情况使用。

另外，类型的定义一般放在index.d.ts这种文件中，d代表declare，也就是说这个文件主要是用来声明类型的，不应该写实际的赋值或者方法的实现等。同时，这个文件还需要在编译时被引入。以本文的vite工程为例，工程中分了node部分和web部分源码，分了两个tsconfig配置，tsconfig.node.json和tsconfig.web.json，这两个文件中也需要引入对应的类型声明文件。


### 问题3：ready-to-show事件在Windows中有时候不执行

启动项目时，日志没有什么异常，但是窗口就是不显示，任务管理器里有进程，也不知道问题出在哪里。经过在各个监听事件打印日志后发现下面的情况：

Electron的Demo中给的代码是监听ready-to-show事件，如下：

```
  mainWindow.on('ready-to-show', () => {
    WLog.log('mainWindow ready-to-show')
    mainWindow.show()
  })
```

但是实际遇到了一些问题，比如引入组件过多时，或者偶尔加了几行Log，再次执行npm run dev时，界面就出不来了。

看了百度和Google上不少内容，最终看到了SOF上有个解决方案，改用了webContents 的 did-finish-load事件触发时进行窗口的显示

https://stackoverflow.com/a/71578179/2979896

```
  mainWindow.webContents.once('did-finish-load', () => {
    WLog.log('mainWindow.webContents did-finish-load')
    mainWindow.show()
  })
```

### 问题4：打包配置

打包配置文件是electron-builder.yml。此文件的配置可以参考Electron-Builder的配置文档。我主要是将oneClick改为了false，并且增加了
```allowToChangeInstallationDirectory: true```可以变更安装目录，否则Windows默认就安装到C盘用户目录AppData里了，C盘越来越大，很难受，很烦这种软件。自己做的软件自然要改一下这种配置。还有很多其他的配置修改点，因为本人暂时还没有深入去折腾，就暂时到此为止了。以后打包Mac版本的时候，再进一步学习和更新一下本文。

## 相关参考资料

1. Electron官方文档

<https://www.electronjs.org/docs/latest/tutorial/quick-start>

2. Electron Forge文档

由于没有采用这个框架，当时只是看了一下。以后再试。

https://www.electronforge.io/

3. Electron-Vite

https://cn.electron-vite.org/guide/introduction

4. NaiveUI

https://www.naiveui.com/zh-CN/dark

5. Electron-Builder文档

最终打包脚本的配置

https://www.electron.build/



