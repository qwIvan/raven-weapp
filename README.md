FORK [YOUZAN](https://github.com/youzan/raven-weapp) 进行npm 封装, 同时自定义后续的功能

#### 安装
yarn add @mqing/sentry-taro-weapp
npm install @mqing/sentry-taro-weapp
#### 初始化Raven

```
Raven.config('https://xxx@crash.youzan.com/x', options).install()
```
在app的onLaunch中初始化(?)，options为可选配置对象，目前支持：
```
options = {
    release: '当前小程序版本'，
    environment: '指定为production才会上报',
    allowDuplicates: true, // 允许相同错误重复上报
    sampleRate: 0.5 // 采样率
}
```
[taro](https://nervjs.github.io/taro/docs/tutorial.html)
```
taro 中为react 的ComponentWillMount(){}
```
#### 收集信息
收集的信息将在上报时被一起带上
##### 基本信息
初始化后raven会默认收集以下信息：
```
{
    SDKversion: '小程序基础库版本',
    WXversion: '微信版本',
    device: '设备型号',
    network: '网络类型',
    system: '系统信息',
}
```
可以通过[Raven.setUserContext(context)](https://docs.sentry.io/learn/context/#capturing-the-user)或者[Raven.setExtraContext(context)](https://docs.sentry.io/learn/context/#extra-context)添加更多信息（kdtId和userId等）
##### 用户行为
###### console
console的行为默认将被自动收集, 注意为附带到breadcrumbs里收集, 见文档或者源码
[breadcrumbs](https://docs.sentry.io/error-reporting/capturing/?platform=browser)
###### ajax
wx.request不可扩展，因此只能手动收集请求的行为：例如在经过封装的请求函数的成功回调内添加：
```
Raven.captureBreadcrumb({
  category: 'ajax',
  data: {
    method: 'get',
    url: 'weapp.showcase.page/1.0.0/get',
    status_code: 200
  }
```
###### dom
ui操作的记录暂不支持
###### location
页面变化的记录暂不支持

#### 信息上报
分为message和exception的上报
##### message
使用[Raven.captureMessage(msg, option)](https://docs.sentry.io/clients/javascript/usage/#capturing-messages)上报需要上报的信息比如ajax的报错等
##### exception
所有uncaught exception都会被小程序捕获封装成msg传递到app的onError中，在onError中上报这些信息：
```
onError(msg) {
    Raven.captureException(msg, {
      level: 'error'
    })
  }
```
Taro:
React 16.0 以上请在OnBoundary 里处理component 错误(**taro 支持**)
