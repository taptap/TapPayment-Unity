# TapPayment-Unity

## 1.概览

### 1.1 TapPayment-Unity 说明
TapPayment 是一项可让您在游戏中销售虚拟产品的服务。<br></br>
您可以在TapPayment中销售以下类型的商品。
- 消耗性商品
- 非消耗性商品

借助 TapTap开发者中心, 您可以方便的创建游戏内商品。<br></br>
并且在游戏中便捷的接入TapPayment的服务，来进行游戏商品的销售。<br></br>
**TapPayment 支付暂时只支持国内支付。支付渠道暂时只支持微信支付 & 支付宝支付**
## 使用前提

### 1.2 接入前准备
#### 1.2.1 开发环境
Unity 2019.4 或更高版本

#### 1.2.2 集成库
使用 TapTap.Payment 前提是必须依赖以下库:
* [TapTap.Bootstrap](https://github.com/TapTap/TapBootstrap-Unity.git)
* [TapTap.Common](https://github.com/TapTap/TapCommon-Unity.git)

## 2. TapPayment-Unity 使用说明

### 2.1 命名空间
```c#
using TapTap.Payment;
```

### 2.2 初始化
定义:
```c#
/**
 * 支付初始化 (会在TapBootstrap的初始化中调用)
 * @param tapConfig TapSDK 初始化参数
 */
void Init(TapConfig tapConfig);
```

示例如下：
```c#
var config = new TapConfig.Builder()
    .ClientID(clientId) // 必须，开发者中心对应 Client ID
    .ClientToken(clientToken) // 必须，开发者中心对应 Client Token
    .ServerURL(serverUrl) // 必须，开发者中心 > 你的游戏 > 游戏服务 > 基本信息 > 域名配置 > API
    .RegionType(RegionType.CN) // 非必须，CN 表示中国大陆，IO 表示其他国家或地区
    .TapPaymentConfig(
        "CN"                   // 地区暂时只支持「中国地区」 
        , "zh_CN"              // 语言暂时只支持中文
        , "https://${domain}" // 微信商户申请H5时提交的授权域名，详见 https://pay.weixin.qq.com/wiki/doc/api/H5.php?chapter=15_4 里 Referer 设置相关部分 
    )
    .ConfigBuilder();
TapBootstrap.Init(config);
```

### 2.3 是否初始化
定义:
```c#
/**
  * 支付模块是否已经初始化
  * @return 
  */
Boolean IsReady();  
```

示例如下：
```c#
var result = TapBootstrap.IsReady();
```

### 2.4 查询单个商品
商品信息定义:
<a name="商品信息"></a>

Parameters | `type` | Description
--- | --- | ---
goodsOpenId | string |DC后台配置的商品id
goodsType | int | 1:消耗型 2:非消耗型
goodsConfig | object | 详见 **商品配置**
goodsPrice | object | 详见 **价格配置**

商品配置定义:
<a name="商品配置"></a>

Parameters | `type` | Description
--- | --- | ---
languageId | string | DC后台 创建商品时配置的 语言ID eg."zh_CN"
goodsName | string |  DC后台 创建商品时配置的 商品名称
goodsDescription | string | DC后台 创建商品时配置的 商品描述

价格配置定义:
<a name="价格配置"></a>

Parameters | 'type' | Description
--- | --- | ---
regionId | string | 国内DC后台的地区ID默认是 "CN"
goodsPriceAmount | string | DC后台创建商品时配置的 商品价格
goodsPriceCurrency | string | 国内DC后台的默认货币 "元"

 `code` | 场景
 --- | ---
 19999 | 服务端返回的数据格式非预期
 50x 的服务器错误 | 服务端内部错误
 401 | 未授权（可以检查初始化配置的参数是否正确配置）
 403 | 用户无权限访问该服务（检查开发者中心是否开通了TapPayment服务）

接口定义
```c#
/**
 * 查询单个商品
 * @param skuId 在DC后台定义的商品id
 * @param action 获取商品回调结果
  */
void QueryProduct(string skuId, Action<SkuDetails, TapError> action);
```

示例如下：
```c#
TapPayment.QueryProduct(skuId, (skuDetail, error) =>
{
    if (error != null)
    {
        // get product fail & do something
    }
    else
    {
        if (skuDetail == null) {
            // not found any product with given skuId
        } else {
            // do something
        }
    }
});
```

### 2.5 查询多个商品
接口定义
```c#
/**
 * 查询多个商品
 * @param skuIds 商品id列表 eg "test01,test02"
 * @param callback 获取商品返回结果
  */
void QueryProducts(string skuIds, Action<SkuDetails, TapError> action);
```

示例如下:
```c#
TapPayment.QueryProducts(skuIds, (list, error) =>
{
    if (error != null)
    {
        // get product fail & do something
    }
    else
    {
        if (list == null || list.Count == 0) {
            // not found any product with given skuIds
        } else {
            // do something
        }
    }
});
```

### 2.5 启动应用内购买

responseCode的定义:
<a name="支付结果"></a>

 `code` | 场景
 --- | ---
 COMPLETE(0) | 购买完成、或者购买被动取消（前端超时）、或者用户点击关闭按钮均会返回该结果
 ERROR(1) |  购买异常，前端返回hash值是fail的情况
 
定义:
```c#
/**
 * 启动购买流程
 * @param skuDetails 购买的商品信息
 * @param roleId 游戏内角色id
 * @param serverId 游戏的服务器id
 * @param extra 游戏的额外信息 json格式的字符串 eg."{\"test\":\"test\"}"
 * @param action
*/
void LaunchBillingFlow(SkuDetails skuDetails, string roleId, String serverId, String extra, Action<int, TapError> action);
```

示例如下:
```c#
TapPayment.LaunchBillingFlow(skuDetailsList[buyPosition], "roleId001", "1e59c66f-51d8-4e9e-afd2-f91965628e9b", "{\"test\":\"test\"}", (responseCode, error) =>
{
    if (error != null)
    {
        // native bridge exception
    }
    else
    {
        if (responseCode == 0)
        {
            // complete
        }
        else if (responseCode == 1)
        {
            // error
        }
    }
});
```


