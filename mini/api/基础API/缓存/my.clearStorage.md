
# 简介
**my.clearStorage** 是清除本地数据缓存的异步接口。

清空内嵌 webview 的存储时不会同时清空当前小程序本身的存储数据。

## 使用限制

- 缓存数据本地加密存储，通过 API 读取时会自动解密返回。
- 覆盖安装支付宝（不是先删除再安装）、关闭小程序，这两种操作均不会导致小程序缓存失效。<br />开发者调用 API 存储的缓存数据，需要自行调用对应的删除/清除 API 进行删除。<br />长期未使用或在应用中心删除的小程序的缓存数据也会被系统清理。
- 小程序缓存默认具有支付宝账号和小程序 ID 两级隔离。
- iOS 客户端支持 iTunes 备份。
- 此 API 支持个人支付宝小程序、企业支付宝小程序使用。

## 扫码体验
![|127x157](https://gw.alipayobjects.com/os/skylark-tools/public/files/99706478534d351cf5e04f615c0cec59.jpeg%26originHeight%3D157%26originWidth%3D127%26size%3D19905%26status%3Ddone%26width%3D127#align=left&display=inline&height=157&margin=%5Bobject%20Object%5D&originHeight=157&originWidth=127&status=done&style=none&width=127)

# 接口调用

## 示例

[小程序在线](https://opendocs.alipay.com/examples/e4f97280-1e31-4262-a4ee-498a786dd4a0) 

### .js 示例代码
```javascript
// .js
my.clearStorage();
```
