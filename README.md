# 微信小程序入坑记录

## 环境相关
- 本地数据mock 手机预览访问 请求本地ip访问地址，需打开调试
- vlc测试直播流地址，如果测试正常，直播组件直播异常可能跟wifi有关，换个wifi试试（如：我公司sit环境wifi测试直播流失败，连接预发pre环境wifi测试正常）
- 设置合法请求域名后，需退出微信开发者工具后重新编译访问

## 自定义组件相关
- hidden属性 针对自定义组件标签不生效，必须包一层view标签，示例中hidden设置无效：
```html
<customComponent hidden="{{true}}"></customComponent>
```
- 自定义组件中的lifetimes配置ready生命周期函数 setTimeout定时器不会执行
```js
Component({
  lifetimes: {
    ready() {
      // 定时器回调不执行
      setTimeout(() => {
        console.log('test')
      }, 100)
    }
  }
})
```
- 直播组件封装在自定义组件中不支持全屏功能
```html
<!-- // 自定义组件模版wxml -->
<live-player id="live-player" src="{{url}}"></live-player>
```
```js
Component({
  data: {
    isFullscreen: false
  },
  ready() {
    this.playerCtx = wx.createLivePlayerContext('live-player')
    // 在自定义组件中尝试全屏不生效
    this.playerCtx.requestFullScreen({
      direction: 90,
      success: () => {
        // this.setData({ isFullscreen: true })
      }
    })
  }
})
```
- data引用类型的字段值改变了，需要通过setData重新设置改字段值，才会引起ui状态变化，示例如下：
```js
import methods from './methods'

Component({
  data: {
    list: []
  },
  properties: {
    pageLiveId: {
      type: Number,
      value: -999
    },
    pageLiveStatus: {
      type: Number,
      value: -999,
      observer(status) {
        if (typeof status !== 'number' || this.data.list.length === 0) {
          return
        }

        const item = this.data.list.find(item => item.liveId === this.properties.pageLiveId)
        if (item && item.streamStatus !== status) {
          item.streamStatus = status

          // console.log(this.data.list)

          // 需要重新设置改变ui状态
          this.setData({
            list: this.data.list
          })
        }
      }
    }
  },
  methods,
  ready() {
    this.init()
  }
})
```

## 其他
- 字符串不支持str.charAt
```js
const str = 'abc'
str.charAt(0) // 不支持
```
- video组件poster属性值必须有协议
```html
<!-- // 必须poster属性值必须带上https协议 -->
<video poster="//wap.kaike.la/test/test.png"></video>
```
```js
Page{
  methods: {
    // 定义一个方法解决
    adjustImgUrl(url) {
      if (url && !url.match(/^http/)) {
        return `https:${url}`
      }
      return url
    },
  }
}
```
- 直播组件内的cover-view cover-image元素 css动画，小程序自身动画 bug，不支持
- 富文本rich-text 装载后台富文本内容（带img标签）需用正则移除行内属性及样式，重设行内样式style="max-width: 100%; width: 100%;"适应移动端布局
```js
Component({
  properties: {
    content: {
      type: String,
      value: "",
      observer(value) {
        this.updateContent(value)
      }
    }
  },

  data: {
    filterContent: ''
  },

  methods: {
    updateContent(value) {
      let content = value

      if (!content) {
        return;
      }

      content = content.replace(/<img[^>]*(src="[^"]+")[^>]*>/gi, (match, $1) => {
        // const src = $1.replace(/src="\/\//, 'src="https://')
        return `<img style="max-width: 100%; width: 100%;" ${$1}>`;
      });

      this.setData({
        filterContent: content
      })
    }
  },

  ready() {
    this.updateContent(this.properties.content)
  }
})

```
- 获取页面路径查询参数 onLoad生命周期函数 参数为query，其他生命周期函数不支持，其他生命周期或者方法，可以通过options属性访问
```js
Page({
  onLoad(query) {
    if (query.liveId) {
      this.init(Number(query.liveId))
    } else {
      this.init()
    }
  },
  onShow() {
    // 通过options访问查询参数
    console.log(this.options.liveId)
  }
})
```


