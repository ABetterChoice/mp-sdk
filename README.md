## 提示

> 在接入前，请先阅读接入前准备
>
> 最新版本：[Releases](https://github.com/ABetterChoice/mp-sdk/releases), 下载小游戏 SDK

---

## 一、集成 SDK

下载微信原生小游戏 SDK，在 game.js 中引入对应的 SDK 文件：
[下载小游戏](https://raw.githubusercontent.com/ABetterChoice/mp-sdk/master/abetterchoice.mg.wx.min.js)

```typescript
const ABetterChoice = require("./abetterchoice.mg.wx.min.js");
```

```typescript
// 1. 初始化配置参数
const config = {
  gameId: "YOUR_GAME_ID", // 项目游戏ID，必选，可以在ABetterChoice平台管理页查看
  apiKey: "YOUR_API_KEY", // 项目API KEY，必选，可以在ABetterChoice平台管理页查看
  autoTrack: {   // 可选，自动采集配置，不设置时默认全部打开
    mgShow: true,  // 自动采集，小程序启动，或从后台进入前台，可选
    mgHide: true,  // 自动采集，小程序从前台进入后台，可选
    mgShare: true, // 自动采集，小程序分享时自动采集，可选
  }
};

// 2. 可选，打开SDK日志，默认关闭，上线前可关闭
ABetterChoice.setLogLevel(0);

// 3. 必须，初始化并启动SDK，必须先调用，确保SDK初始化成功后再执行功能方法
ABetterChoice.init(config).then((initResult) => {
   console.log('初始化结果：' + initResult)

   // 4. 必须，用户的登录唯一标识，此数据对应上报数据里的user_id，此时user_id的值为ABC
   ABetterChoice.login('ABC');
};

```

配置对象**Config**其他可选参数说明：

- **serverUrl**：可选，数据上报地址域名，默认是 https://data.abetterchoice.cn/。
- **attributes**: 可选，实验分流属性条件使用，类型为{ string : string[] }，其中 string 为条件属性，string[]为对应的条件属性值数组
- **enableAutoExposure**：可选，实验分流使用，不设置时默认值为 true。如果设置为 false，当调用 AB 实验分流时，曝光数据将关闭自动上报。
- **enableAutoPoll**：可选，实验分流使用，默认值为 true。如果设置为 true，实验和功能标志数据将每 10 分钟轮询并更新。

**警示**：无论您获取帐号 ID 是异步还是同步的，请在使用 SDK 接入完成后用下面的*login*接口进行帐号 ID 的登陆，以确保数据计算结果的准确性。

```typescript
// 用户的唯一标识，此数据对应上报数据里的user_id，此时user_id的值为ABC，不登陆会影响数据计算结果的准确性
ABetterChoice.login("ABC");
```

**注意**：

> 在上报数据与实验分流之前，请先在微信公众平台或其他平台的开发设置中，将下面默认请求 URL 加入到服务器域名的 request 列表中，主要有：
>
> 分流 URL：https://mobile.abetterchoice.cn
>
> 数据上报 URL：https://data.abetterchoice.cn
>
> 若您使用的是私有化部署版本，请与运维同学确认上报地址

## 二、常用功能

在使用常用功能之前，确保 SDK 已初始化成功并已登陆帐号 ID

### 2.1 设置帐号 ID

在用户进行登录时，可调用 `login` 来设置用户的账号 ID， SDK 将会以账号 ID 作为身份识别 ID，并且设置的账号 ID 将会一直保留，多次调用 `login` 将覆盖先前的账号 ID。

```typescript
// 用户的唯一标识，此数据对应上报数据里的user_id，此时user_id的值为ABC
ABetterChoice.login("ABC");
```

### 2.2 设置公共属性

公共事件属性指的就是每个事件都会带有的属性，您可以调用 `setCommonProperties` 来设置公共事件属性，我们推荐您在发送事件前，先设置公共事件属性。对于一些重要的属性，譬如用户的会员等级、来源渠道等，这些属性需要设置在每个事件中，此时您可以将这些属性设置为公共事件属性。

```typescript
var commonProperties = {
  vipSource: "ABC", //字符串
  vipLevel: 1, //数字
  isVip: true, //布尔
  birthday: new Date(), //对象
  object: { key: "value" }, //对象
  object_arr: [{ key: "value" }], //对象组
  arr: ["value"], //数组
};
// 设置公共属性
ABetterChoice.setCommonProperties(commonProperties);
```

公共事件属性将会被保存到缓存中，无需每次启动时调用。如果调用 `setCommonProperties`上传了先前已设置过的公共事件属性，则会覆盖之前的属性。

- Key 为该属性的名称，为字符串类型，规定只能以字母开头，包含数字，字母和下划线 "\_"，长度最大为 50 个字符。
- Value 为该属性的值，目前仅支持字符串。

### 2.3 发送事件

您可以调用 `track` 来上传事件，建议您根据先前梳理的埋点文档来设置事件的属性以及发送信息的条件，此处以用户购买某商品作为范例：

```typescript
ABetterChoice.track({
  eventName: "product_buy", // 事件名称
  properties: {
    product_name: "商品名",
  }, //事件属性
});
```

### 2.4 获取 AB 实验

```typescript
// 获取实验分流信息，默认会在获取分流信息同时会进行自动上报。
const experiment = ABetterChoice.getExperiment("abc_layer_name");
if (experiment === undefined) {
  // 无命中，执行默认版本
  handleView("无命中实验，执行默认版本");
  return;
}
// 现在您可以获取参数并在代码中直接使用这些参数。
const shouldShowBanner = experiment.getBoolValue("should_show_banner", true);
```

### 2.5 实验曝光

```typescript
// 当设置enableAutoExposure为false时，您可以根据上面获取的实验分流信息进行手动记录曝光
ABetterChoice.logExperimentExposure(experiment);
```

### 2.6 获取配置开关

```typescript
// 获取配置开关参数名称为：new_feature_flag的配置开关值信息
const configInfo = ABetterChoice.getConfig("new_feature_flag");
// 获取对应配置开关的参数值,其中参数为默认值
const boolValue = configInfo?.getBoolValue(false);
// 其它根据平台配置的值类型进而获取不同的类型值
// const stringValue = configInfo?.getStringValue('banner');
// const numberValue = configInfo?.getNumberValue(1000);
```

若配置了条件，如下图创建所示，假设您创造了开关配置参数名称为'new_feature_flag'，配置条件配置参数属性为'city'与'age'，条件参数属性值为'shenzhen'与'18'，则满足这个条件则会进行下发布尔值 true。

![IMG](https://cdn.abetterchoice.cn/static/cms/5640e1e9ac.jpeg)

```typescript
// 初始化SDK时需配置条件属性：attributes
var config = {
  .......
  .......
  // 配置条件属性
  attributes: {
    city: ['shenzhen'],
    age: ['18']
  }
}
......
......
// 获取配置开关参数名称为：new_feature_flag的配置开关值信息
const configInfo = ABetterChoice.getConfig("new_feature_flag");
// 获取对应配置开关的参数值,其中参数为默认值
const boolValue = configInfo?.getBoolValue(false);
```

## 三、最佳实践

[下载小游戏 SDK (下载小游戏)](https://download.thinkingdata.cn/client/release/ta_mg_sdk.zip), 在 game.js 中引入对应的 SDK 文件，引入 SDK 之后，您就可以创建 SDK 实例，开始上报数据了：

```typescript
// 初始化配置参数
const config = {
  gameId: "YOUR_GAME_ID", //项目游戏ID，必选，可以在ABetterChoice平台管理页查看
  apiKey: "YOUR_API_KEY"，//项目API KEY，必选，可以在ABetterChoice平台管理页查看
  autoTrack: {   // 可选，自动采集配置，不设置时默认全部打开
    mgShow: true,  // 自动采集，小程序启动，或从后台进入前台
    mgHide: true,  // 自动采集，小程序从前台进入后台
    mgShare: true, // 自动采集，小程序分享时自动采集
  }
};
// 打开SDK日志，默认关闭，上线前可关闭
ABetterChoice.setLogLevel(0);
// 初始化并启动SDK
await initResult = ABetterChoice.init(config);
if (initResult) {
  console.log('初始化结果：' + initResult);
}
// 用户的登录帐号唯一标识，此数据对应上报数据里的user_id，此时user_id的值为ABC，不登陆会影响数据计算结果的准确性
ABetterChoice.login('ABC');

// 设置公共事件属性
const commonProperties = {
    channel : "ta", //字符串
    age : 1,//数字
    isSuccess : true,//布尔
    birthday :  new Date(),//对象
    object : { key : "value" },//对象
    object_arr : [ { key : "value" } ],//对象组
    arr : [ "value" ]//数组
};
ABetterChoice.setCommonProperties(commonProperties);

// 清除公共事件属性
ABetterChoice.clearCommonProperties();

//发送事件
ABetterChoice.track(
    eventName: "product_buy", // 事件名称
    properties: {
        product_name: "商品名"
    } //事件属性
);

// 获取实验分流信息
const experiment = ABetterChoice.getExperiment('abc_layer_name');
// 获取实验配置的参数值并在代码中选择直接使用这些参数。
const shouldShowBanner = experiment.getBoolValue("should_show_banner", true);
if (shouldShowBanner) {
  handleView('执行显示Banner业务逻辑');
} else {
  handleView('执行不显示Banner业务逻辑');
}
// 根据上面获取到的实验信息主动进行曝光
ABetterChoice.logExperimentExposure(experiment);

// 获取配置开关名为：new_feature_flag的配置开关值信息
const configObj = ABetterChoice.getConfig("new_feature_flag");
```
