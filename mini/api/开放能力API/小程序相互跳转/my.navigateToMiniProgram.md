# 简介

**my.navigateToMiniProgram** 是用于跳转到其它小程序的 API。

如需跳转到目标小程序的指定开发版本，可查看 [联调设置](https://opendocs.alipay.com/mini/ide/integration-testing)。

有关小程序跳转的更多知识，可查看 [小程序相互跳转 FAQ](https://opendocs.alipay.com/mini/api/xqvxl4)。 

## 使用限制

此 API 支持个人支付宝小程序、企业支付宝小程序使用。

# 接口调用

## 示例代码

```javascript
// 跳转小程序示例代码
my.navigateToMiniProgram({
  appId: 'xxxxxxxxxxxxxxx', // 16 位数字
  path: 'pages/index/index?p=1&q=2', 
  query: {
    a: 'foo',
    b: 'bar',
  },
  extraData: {
    x: {
      y: 'z'
    },
  },
  success: () => {
    console.log('navigateToMiniProgram success');
  },
  fail: (res) => {
    my.alert({
      title: 'navigateToMiniProgram fail',
      content: JSON.stringify(res),
    });
  }
});

// 目标小程序获取页面参数
Page({
  onLoad(query) {
    console.log('query:', query); // query: { p: 1, q: 2 }
  },
});

// 目标小程序获取应用参数
App({
  onLaunch(options) {
    const { query, referrerInfo: { extraData } = {} } = options;
    console.log('query:', query); // query: { a: 'foo', b: 'bar' }
    console.log('extraData:', extraData); // extraData: { x: { y: 'z' } }
  },
  onSomeButtonTap() {
    const { query, referrerInfo: { extraData } = {} } = my.getLaunchOptionsSync();
    console.log('query:', query); // query: { a: 'foo', b: 'bar' }
    console.log('extraData:', extraData); // extraData: { x: { y: 'z' } }
  },
});

```

## 入参
Object 类型，参数如下：

| **参数** | **类型** | **必填** | **描述** |
| --- | --- | --- | --- |
| appId | String | 是 | 要跳转的目标小程序 appId。 |
| path | String | 否 | 要打开的目标小程序页面路径。省略此参数则打开目标小程序首页。<br />path 中`?`后的内容会被解析为页面的 query 参数，目标小程序可在 `Page.onLoad(query)` 中获取。参数解析规则可参考 [小程序全局 / 页面参数设置以及解析细节](https://opendocs.alipay.com/mini/03durs)。 |
| query | Object | 否 | 传给目标小程序启动参数的 query 字段，目标小程序可在 `App.onLaunch(options)` 或 `App.onShow(options)` 中通过 options.query 获取，也可通过 my.getLaunchOptionsSync().query 获取。<br />基础库从 2.7.19 开始支持此参数，使用 1.x 版本的小程序需要 [升级基础库](https://opendocs.alipay.com/mini/framework/lib-upgrade-v2)。 |
| extraData | Object | 否 | 传给目标小程序的额外数据。<br />目标小程序可在 `App.onLaunch(options)` 或 `App.onShow(options)` 中通过 options.referrerInfo?.extraData 获取。 |
| success | Function | 否 | 调用成功的回调函数。|
| fail | Function | 否 | 调用失败的回调函数。 |
| complete | Function | 否 | 调用结束的回调函数（调用成功、失败都会执行）。 |

## 错误码
| **错误码** | **描述** | **解决方案** |
| --- | --- | --- |
| 30 | 目标应用不允许被跳转。 | 请目标小程序检查设置。操作路径：[开放平台控制台](https://open.alipay.com/dev/workspace) > 点击要设置的小程序，进入小程序详情页 > **设置** > **基础设置** > **小程序互相跳转** 中设置 **允许所有小程序跳转** 或 **指定小程序跳转**。 |
| 31 | 跳转失败。 | 检查 appId 参数。appId 为 16 位数字，由开放平台在小程序创建时分配。<br>注意：如果目标小程序未上架，调用本接口仍会触发 success 回调而非失败报错，未通过 [联调设置](https://opendocs.alipay.com/mini/ide/integration-testing) 指定跳转开发版的用户将看到 “暂未找到此功能” 提示页。 |

# 常见问题 FAQ

## Q：拿到目标小程序的 scheme （以 alipays:// 开头），应该如何跳转？
A：scheme 是用于从外部应用打开支付宝，在小程序内部一般不能直接使用。如果 scheme 中 appId 是 16 位，且只包含 page、query 参数，则应转换成 my.navigateMiniProgram 调用；其他情况（appId 为 8 位，或者有额外参数），需使用 [my.ap.navigateToAlipayPage](https://opendocs.alipay.com/mini/api/navigatetoalipaypage) 跳转，并联系合作的支付宝业务人员申请加入白名单。参考转换逻辑如下：
```javascript
// 将 scheme 转换成 navigateMiniProgram 可接受的参数
// 注意：此代码主要用来说明转换逻辑，可作为线下工具使用，不建议直接用于线上
function schemeToParams(scheme) {
  var parseQuery = (str) => {
    var ret = {};
    str.split('&').forEach(s => {
      var p = s.includes('=') ? s.indexOf('=') : s.length;
      ret[decodeURIComponent(s.slice(0, p))] = decodeURIComponent(s.slice(p + 1));
    });
    return ret;
  };
  var params = {};
  var query = parseQuery(scheme.slice(scheme.indexOf('?') + 1));
  for (var k in query) {
    var v = query[k];
    if (k == 'appId') {
      if (v.length != 16) {
        console.log(`! 非 16 位 appId '${v}'`);
        params = null;
        break;
      }
    } else if (k == 'page') {
      k = 'path';
    } else if (k == 'query') {
      v = parseQuery(v);
    } else {
      console.log(`! 不支持参数 '${k}' `);
      params = null;
      break;
    }
    params[k] = v;
  }
  if (!params) {
    console.log(`! 请使用 my.ap.navigateToAlipayPage() 跳转 '${scheme}'`);
  }
  return params;
}

// 使用示例
var scheme = 'alipays://platformapi/startapp?appId=2022061812345678&page=%2Fpages%2Findex%2Findex&query=foo%3Dbar';
console.log(schemeToParams(scheme));
```

## Q：my.navigateMiniProgram 的调用参数如何转换为等价的 scheme ？
A：如果调用参数只含 appId、path、query，则可以转换成等价的 scheme。extraData 不支持在 scheme 中使用。参考转换逻辑如下：
```javascript
// 将 my.navigateMiniProgram 的参数转换成 scheme
// 注意：此代码主要用来说明转换逻辑，可作为线下工具使用，不建议直接用于线上
function paramsToScheme(params) {
  var { appId, path, query, extraData } = params;
  var scheme = `alipays://platformapi/startapp?appId=${appId}`;
  if (path) {
    scheme += `&page=${encodeURIComponent(path)}`;
  }
  if (query) {
    query = Object.keys(query).map(k => `${encodeURIComponent(k)}=${encodeURIComponent(query[k])}`).join('&');
    scheme += `&query=${encodeURIComponent(query)}`;
  }
  if (extraData && Object.keys(extraData).length) {
    console.log('! extraData 不支持在 scheme 中使用', params);
  }
  return scheme;
}

// 使用示例
var params = {
  appId: '2022061812345678',
  path: '/pages/index/index',
  query: { foo: 'bar' }
};
console.log(paramsToScheme(params));
```

# 附录：部分官方小程序 appId 列表

在小程序中跳转官方业务，请先查看官方公开的 [appCode 列表](https://opendocs.alipay.com/mini/api/navigatetoalipaypage#String%20appCode) 并使用 my.ap.navigateToAlipayPage 跳转；如果 appCode 中未找到，再参考下表:

| **小程序** | **appId** | **描述** |
| --- | --- | --- |
| 我的小程序 | 2018072560844004 | 支付宝小程序官方出品，与支付宝用户的小程序相关的内容和服务都在这。 |
| 集分宝 | 2019092567759928 | 集分宝的官方小程序。 |
| 支付宝校园缴费 | 2021002140649424 | 支付宝教育缴费是由支付宝推出的一款平台型的教育缴费服务。 |

如果 appCode 列表及上表均未包含所需跳转的官方小程序，并确实有强需求，可联系合作的支付宝业务人员。
