
# XPay聚合支付系统接口文档

## 支付流程
### 非WAP支付方式支付
支付交易场景为商户需要在手机应用( APP )，可以选择 XPay SDK 中的支付 Charge 功能（非wap支付方式）， XPay 支付 Charge 功能需要同时使用客户端 SDK 和服务端 SDK（目前只开放接口）。

1. 用户在客户端选择商品并提交订单，客户端需要向你的服务端传递支付要素。注意：XPay SDK 不涉及你的客户端和你的服务端之间的数据交互，此处请你自定义通信方式。
2. 服务端接收到客户端请求参数，并调用 Server-SDK(目前接口)封装的创建支付 Charge 的方法请求 XPay 。
3. XPay 响应你的服务端请求，返回 Charge (支付凭据)给你的服务端。
4. 你的服务端响应你的客户端请求，需要将该 Charge 对象完整的返回给你的客户端，注意：这里的 Charge 返回类型必须是 JSON 格式。
5. 客户端拿到支付凭据 Charge 对象后，需要调用 Client-SDK 封装的方法调起支付控件，用户完成支付。
6. 第三方支付渠道会直接在客户端返回支付结果，此处不要使用客户端的成功结果更新订单的最终状态。
7. 在 XPay 管理平台配置 Webhooks 的 charge.succeeded 事件。支付完成时，XPay 会主动以 POST 方式向你配置在管理平台上的 Webhooks 通知地址发送支付结果，建议订单状态的更新对比客户端的渠道同步回调信息和服务端的 XPay Webhooks 通知来确定是否修改。
8. 同时，建议在处理逻辑中添加主动查询机制：如果在可接受的时间范围内没有收到 Webhooks 通知，你也可以调用 Server-SDK 封装的查询方法，主动向 XPay 发起请求来获得订单状态，该查询结果可以作为交易结果。

### WAP支付方式
支付交易场景为商户需要在手机应用( APP )，可以选择 XPay SDK 中的支付 Charge 功能（WAP支付方式）， XPay 支付 Charge 功能需要同时使用客户端 SDK 和服务端 SDK（目前只开放接口）。

1. 用户在客户端选择商品并提交订单，客户端需要向你的服务端传递支付要素。注意：XPay SDK 不涉及你的客户端和你的服务端之间的数据交互，此处请你自定义通信方式。
2. 服务端接收到客户端请求参数，并调用 Server-SDK(目前接口)封装的创建支付 Charge 的方法请求 XPay 进行预下单。
3. XPay 响应你的服务端请求，返回 Charge (预支付凭据)给你的服务端。
4. 你的服务端响应你的客户端请求，需要将该 Charge 对象完整的返回给你的客户端，注意：这里的 Charge 返回类型必须是 JSON 格式。
5. 客户端拿到支付凭据 Charge 对象后，需要调用 Client-SDK 封装的方法调起H5选择订单支付方式。
6. 当选择对应支付方式后，H5会向 XPay 服务端发起该订单流水号的支付请求，并返回对应支付方式的支付凭证给H5。H5调起第三方支付应用完成支付流程。
7. 第三方支付渠道会异步通知 XPay 服务端，XPay 通知H5更新支付结果并异步通知商家服务端。
8. 同时，建议在处理逻辑中添加主动查询机制：如果在可接受的时间范围内没有收到 Webhooks 通知，你也可以调用 Server-SDK 封装的查询方法，主动向 XPay 发起请求来获得订单状态，该查询结果可以作为交易结果。

## 接口文档

XPay API 采用 REST 风格设计。所有接口请求地址都是可预期的以及面向资源的。使用规范的 HTTP 响应代码来表示请求结果的正确或错误信息。使用 HTTP 内置的特性，如 HTTP Authentication 和 HTTP 请求方法让接口易于理解。所有的 API 请求都会以规范友好的 JSON 对象格式返回（包括错误信息）。

### 基础URL

http://api.staging.xpay.ai/{版本}/{接口}

版本： v1

### 认证

在调用 API 时，必须提供 API Key 作为每个请求的身份验证。你可以在管理平台内管理你的 API Key。API Key 是商户在 XPay 系统中的身份标识，请安全存储，确保其不要被泄露。如需获取或更新 API Key ，也可以在管理平台的「企业设置」->「开发设置」内进行操作。

XPay API 使用 HTTP Basic Auth 进行认证。 将 API Key 作为 basic auth 的用户名。不需要填写密码。

* 本文档所有操作的api都要在header里加上Authorization字段作请求,value为``“Basic +Base64(api_secret+':')”``

### 测试key

### 服务器错误码

* **200** OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
* **201** CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
* **202** Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
* **204** NO CONTENT - [DELETE]：用户删除数据成功。
* **400** INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
* **401** Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
* **403** Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
* **404** NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
* **406** Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
* **410** Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
* **422** Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
* **500** INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。

所有400~500之间的错误，服务端都会返回
400 ~ 500

```json
{
    "error": {
        "type": "auth_error",//错误类型，可以是  invalid_request_error 、 api_error 、 channel_error 或  card_error 。
        "message": "找不到用户,请重新登录!",//返回具体的错误描述。
        "code": "auth_fail"//错误码，由第三方支付渠道返回的错误代码。
    }
}
```

### 交易

你可以创建一个 `charge` 对象向用户收款。 `charge` 是一个支付凭据对象，所有和支付相关的要素信息都存储在这个对象中，你的服务端可以通过发起支付请求来创建一个新的 `charge` 对象，也可以随时查询一个或者多个 `charge` 对象的状态。每个 `charge` 对象都拥有一个标识 `id`，该 `id` 在 XPay 系统内唯一。

属性

| 属性名 | 类型 | 描述 |
| ------ | ---- | ---- |
| id | String | 由 XPay 生成的支付对象 ID， 27 位字符串。 |
| object | String | 值为 "charge"。|
| created | timestamp | 支付创建时的 Unix 时间戳。|
| livemode | boolean | 是否处于  live 模式。|
| paid | boolean | 是否已付款。|
| refunded | boolean | 是否存在退款信息，无论退款是否成功。|
| app | object | 支付使用的  app 对象，必须填写对象的id 例如：{"id":"xxxxx"}|
| channel | string | 支付使用的第三方支付渠道（微信：wx、支付宝：alipay）|
| order_no | string | 商户订单号，适配每个渠道对此参数的要求，必须在商户系统内唯一。( alipay : 1-64 位，  wx : 2-32 位， bfb : 1-20 位， upacp : 8-40 位， yeepay_wap :1-50 位， jdpay_wap :1-30 位， qpay :1-30 位， cmb_wallet :10 位纯数字字符串。注：除  cmb_wallet 外的其他渠道推荐使用 8-20 位，要求数字或字母，不允许特殊字符)。|
| client_ip | string | 发起支付请求客户端的 IP 地址，格式为 IPv4 整型，如 127.0.0.1。|
| amount | int | 订单总金额（必须大于0），单位为对应币种的最小货币单位，人民币为分。如订单总金额为 1 元， amount 为 100，么么贷商户请查看申请的借贷金额范围。|
| subject | string | 商品标题，该参数最长为 32 个 Unicode 字符，银联全渠道（ upacp / upacp_wap ）限制在 32 个字节。|
| body | string | 商品描述信息，该参数最长为 128 个 Unicode 字符， yeepay_wap 对于该参数长度限制为 100 个 Unicode 字符。|
| extra | object | 特定渠道发起交易时需要的额外参数，以及部分渠道支付成功返回的额外参数，详细参考 [支付渠道 extra 参数说明](https://github.com/xpayai/XPayDocument/blob/master/%E6%94%AF%E4%BB%98%E6%B8%A0%E9%81%93%20extra%20%E5%8F%82%E6%95%B0%E8%AF%B4%E6%98%8E.md) 。|
| time_paid | timestamp | 订单支付完成时的 Unix 时间戳。（银联支付成功时间为接收异步通知的时间）|
| time_expire | timestamp | 订单失效时的 Unix 时间戳。时间范围在订单创建后的 1 分钟到 15 天，默认为 1 天，创建时间以 XPay 服务器时间为准。 微信对该参数的有效值限制为 2 小时内；银联对该参数的有效值限制为 1 小时内。|
| transaction_no | string | 支付渠道返回的交易流水号。|
| failure_code | string | 订单的错误码，详见 错误 中的错误码描述。|
| failure_msg | string | 订单的错误消息的描述。|
| credential | object | 支付凭证，用于客户端发起支付。|
| description | string | 订单附加说明，最多 255 个 Unicode 字符。|

**目前支持的channel：**

| 支付渠道字符串 | 描述 |
| ------ | ---- |
| wx | 微信App支付 |
| alipay | 支付宝App支付 |
| huanxun | pc银联支付 |
| huanxun_mobile | 移动银联支付 |
| huanxun_ali_scan | 支付宝扫码 |
| huanxun_wx_scan | 微信扫码 |
| huanxun_mobile | 移动银联支付 |
| wap_zhimou | 智眸WAP支付 |

#### 创建 Charge 对象

发起一次支付请求时需要创建一个新的 charge 对象，获取一个可用的支付凭据用于客户端向第三方渠道发起支付请求。当支付成功后，XPay 会发送 Webhooks 通知。

**POST** /charges

**请求参数:**

| 属性名 | 类型 | 描述 | 必填 |
| ------ | ---- | ---- | ---- |
| order_no | string | 商户订单号，适配每个渠道对此参数的要求，必须在商户系统内唯一。( alipay : 1-64 位，  wx : 2-32 位， bfb : 1-20 位， upacp : 8-40 位， yeepay_wap :1-50 位， jdpay_wap :1-30 位， qpay :1-30 位， cmb_wallet :10 位纯数字字符串。注：除  cmb_wallet 外的其他渠道推荐使用 8-20 位，要求数字或字母，不允许特殊字符)。| 是 |
| app | string | 支付使用的  app 对象的  id| 是 |
| channel | string | 支付使用的第三方支付渠道（微信：wx、支付宝：alipay）| 是 |
| amount | int | 订单总金额（必须大于0），单位为对应币种的最小货币单位，人民币为分。如订单总金额为 1 元， amount 为 100，么么贷商户请查看申请的借贷金额范围。| 是 |
| client_ip | string | 发起支付请求客户端的 IP 地址，格式为 IPv4 整型，如 127.0.0.1。| 是 |
| subject | string | 商品标题，该参数最长为 32 个 Unicode 字符，银联全渠道（ upacp / upacp_wap ）限制在 32 个字节。| 是 |
| body | string | 商品描述信息，该参数最长为 128 个 Unicode 字符， yeepay_wap 对于该参数长度限制为 100 个 Unicode 字符。| 是 |
| extra | object | 特定渠道发起交易时需要的额外参数，以及部分渠道支付成功返回的额外参数，详细参考 支付渠道 extra 参数说明 。| 否 |
| time_expire | timestamp | 订单失效时的 Unix 时间戳。时间范围在订单创建后的 1 分钟到 15 天，默认为 1 天，创建时间以 XPay 服务器时间为准。 微信对该参数的有效值限制为 2 小时内；银联对该参数的有效值限制为 1 小时内。| 否 |
| metadata | object | 用户指定的 metadata 参数。你可以使用键值对的形式来构建自己的 metadata | 否 |
| description | string | 订单附加说明，最多 255 个 Unicode 字符。| 否 |

**请求示例:**

```
XPay.api_key = "sk_test_ibbTe5jLGCi5rzfH4OqPW9KC"
XPay::Charge.create(
   :subject  => "Your Subject",
   :body     => "Your Body",
   :amount   => 100,
   :order_no => "20170704170645918084",
   :channel  => "wx",
   :client_ip=> '127.0.0.1',
   :app => {'id' => "app_1Gqj58ynP0mHeX1q"}
 )
```
**返回示例:**

```
{
    "id":"ch_aiaQ9rBuULCxUwGedisf2Vkf",
    "object":"charge",
    "created":1499159204,
    "livemode":"true",
    "paid":"false",
    "app":"app_1Gqj58ynP0mHeX1q",
    "channel":"wx",
    "order_no":"20170704170645918084",
    "client_ip":"127.0.0.1",
    "amount": 100,
    "currency":"cny",
    "subject":"Your Subject",
    "body":"Your Body",
    "time_paid":null,
    "time_expire":null,
    "transaction_no":null,
    "failure_code":null,
    "failure_msg":null,
    "credential":{
        "object":"credential",
        "wx":{
            "appid":"xxxxxxxxxxxx",
            "partnerid":"100000000",
            "prepayid":"wx20170704170644a20d47b15a0172580567",
            "package":"Sign=WXPay",
            "noncestr":"myrmIOLe3Ud84W4N",
            "timestamp":"1499159204",
            "sign":"D0ECC58B504FA7F8CF7E95114DD56E5C"
        }
    },
    “metadata”:null,
    "description":null
}
```

#### 查询 Charge 对象

你可以在后台异步通知之前，通过查询接口确认支付状态。通过charge对象的id查询一个已创建的charge对象。

**GET** /charges/:id

**请求参数:**

| 属性名 | 类型 | 描述 | 必填 |
| ------ | ---- | ---- | ---- |
| id | string | 查询的  charge 对象  id 。 | 是 |

**请求返回：**

```
{
    "id":"ch_aiaQ9rBuULCxUwGedisf2Vkf",
    "object":"charge",
    "created":1499159204,
    "livemode":"true",
    "paid":"false",
    "app":"app_1Gqj58ynP0mHeX1q",
    "channel":"wx",
    "order_no":"20170704170645918084",
    "client_ip":"127.0.0.1",
    "amount": 100,
    "currency":"cny",
    "subject":"Your Subject",
    "body":"Your Body",
    "time_paid":null,
    "time_expire":null,
    "transaction_no":null,
    "failure_code":null,
    "failure_msg":null,
    "credential":{
        "object":"credential",
        "wx":{
            "appid":"xxxxxxxxxxxx",
            "partnerid":"100000000",
            "prepayid":"wx20170704170644a20d47b15a0172580567",
            "package":"Sign=WXPay",
            "noncestr":"myrmIOLe3Ud84W4N",
            "timestamp":"1499159204",
            "sign":"D0ECC58B504FA7F8CF7E95114DD56E5C"
        }
    },
    "description":null
}
```

### Webhooks回调

#### Events事件

为了便于客户系统或者第三方系统处理客户的交易信息，XPay 系统支持 Webhooks 功能，可以按照客户要求把特定的事件结果推送到指定的地址以便于客户做后续处理。目前支持的事件有支付成功后回调。 以下是关于接收 Webhooks 通知的说明：

* Webhooks 是 XPay 和你服务器间的交互，相比页面跳转同步通知可以在页面上显示出来，这种交互方式是不可见的。
* Webhooks 通知是以 POST 形式发送的 JSON ，放在请求的 body 里，内容是 Event 对象。
* 你需要监听并接收 Webhooks 通知，接收到 Webhooks 后需要返回服务器状态码 2xx 表示接收成功，否则请返回状态码 500。
* 若返回的服务器状态码不是 2xx，XPay 服务器会在 25 小时内向你的服务器不断重发通知，最多 10 次。Webhooks 首次是即时推送，重试通知时间间隔为 5s、10s、2min、5min、10min、30min、1h、2h、6h、15h，直到你正确回复状态 2xx 或者超过最大重发次数，XPay 将不再发送。

其中 Event 对象属性定义如下。

| 属性名 | 类型 | 描述 |
| ------ | ---- | ---- |
| id | string | 事件对象  id ，由 XPay 生成，28 位长度字符串。 |
| object | string | 值为 "event"。 |
| livemode | boolean | 事件是否发生在生产环境。 |
| created | timestamp | 事件发生的时间。 |
| pending_webhooks | int | 推送未成功的 webhooks 数量。 |
| type  | string | 事件类型，参考事件类型 |
| request  | string | API Request ID。值 "null" 表示该事件不是由 API 请求触发的。 |
| data  | hash | 绑定在事件上的数据对象 |

Event 事件类型

每个事件类型的作用域都是单个应用，目前支持的事件包括：

| 事件类型 | 描述 |
| ------ | ---- |
| charge.succeeded | 支付对象，支付成功时触发。 |

**charge.succeeded事件示例：**

```
{
    "id": "evt_ugB6x3K43D16wXCcqbplWAJo",
    "created": 1427555101,
    "livemode": true,
    "type": "charge.succeeded",
    "data": {
        "object": {
            "id": "ch_Xsr7u35O3m1Gw4ed2ODmi4Lw",
            "object": "charge",
            "created": 1427555076,
            "livemode": true,
            "paid": true,
            "refunded": false,
            "app": "app_1Gqj58ynP0mHeX1q",
            "channel": "upacp",
            "order_no": "123456789",
            "client_ip": "127.0.0.1",
            "amount": 100,
            "amount_settle": 100,
            "currency": "cny",
            "subject": "Your Subject",
            "body": "Your Body",
            "extra": {},
            "time_paid": 1427555101,
            "time_expire": 1427641476,
            "time_settle": null,
            "transaction_no": "1224524301201505066067849274",
            "amount_refunded": 0,
            "failure_code": null,
            "failure_msg": null,
            "metadata": {},
            "credential": {},
            "description": null
        }
    },
    "object": "event",
    "pending_webhooks": 0,
    "request": "iar_qH4y1KbTy5eLGm1uHSTS00s"
}
```

## Webhooks使用指南

为了便于客户系统或者第三方系统处理客户的交易信息，XPay 系统支持 Webhooks 功能，可以按照客户要求把特定的事件结果推送到指定的地址以便于客户做后续处理。目前支持的事件包括支付结果、和退款结果。

### 配置 Webhooks
在商家后台菜单栏选择 webhooks 后，设置接收 Webhooks 事件的地址和事件类型。

### 接收 Webhooks 通知

1. Webhooks 是 XPay 和你服务器间的交互，不像页面跳转同步通知可以在页面上显示出来，这种交互方式是不可见的。
2. Webhooks 通知是以 POST 形式发送的 JSON ，放在请求的 body 里，内容是 Event 对象。
3. 你需要监听并接收 Webhooks 通知，接收到 Webhooks 后需要返回服务器状态码 ``2xx ``表示接收成功，否则请返回状态码 ``500``。
4. 若你的服务器未正确返回 2xx，XPay 服务器会在 25 小时内向你的服务器不断重发通知，最多 10 次。Webhooks 首次是即时推送，重试通知时间间隔为 5s、10s、2min、5min、10min、30min、1h、2h、6h、15h，直到你正确回复状态 2xx 或者超过最大重发次数，XPay 将不再发送。

### 验证 Webhooks 签名（可选）

为了进一步确保安全性， XPay Webhooks 提供了签名验证。你需要验证该通知是来自于 XPay 后再更新你的订单状态

#### 签名简介

XPay 的 Webhooks 通知包含了签名字段，可以使用该签名验证 Webhooks 通知的合法性。签名放置在 header 的自定义字段 XPay-Signature 中，签名用 RSA 私钥对 Webhooks 通知使用 RSA-SHA256 算法进行签名，以 base64 格式输出。

#### 验证签名

XPay 在管理平台中提供了 RSA 公钥，供验证签名，该公钥具体获取路径：点击管理平台右上角公司名称->企业设置-> XPay 公钥。验证签名需要以下几步：

从 header 取出签名字段并对其进行 base64 解码。
获取 Webhooks 请求的原始数据。
将获取到的 Webhooks 通知、 XPay 管理平台提供的 RSA 公钥、和 base64 解码后的签名三者一同放入 RSA 的签名函数中进行非对称的签名运算，来判断签名是否验证通过。可参考以下java代码：

```
/**
     * 获得公钥
     * @return
     * @throws Exception
     */
    public static PublicKey getPubKey() throws Exception {
        String pubKeyString = getStringFromFile(pubKeyPath);
        pubKeyString = pubKeyString.replaceAll("(-+BEGIN PUBLIC KEY-+\\r?\\n|-+END PUBLIC KEY-+\\r?\\n?)", "");
        byte[] keyBytes = Base64.decodeBase64(pubKeyString);

        // generate public key
        X509EncodedKeySpec spec = new X509EncodedKeySpec(keyBytes);
        KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        PublicKey publicKey = keyFactory.generatePublic(spec);
        return publicKey;
    }

    /**
     * 验证签名
     * @param dataString
     * @param signatureString
     * @param publicKey
     * @return
     * @throws NoSuchAlgorithmException
     * @throws InvalidKeyException
     * @throws SignatureException
     */
    public static boolean verifyData(String dataString, String signatureString, PublicKey publicKey)
            throws NoSuchAlgorithmException, InvalidKeyException, SignatureException, UnsupportedEncodingException {
        byte[] signatureBytes = Base64.decodeBase64(signatureString);
        Signature signature = Signature.getInstance("SHA256withRSA");
        signature.initVerify(publicKey);
        signature.update(dataString.getBytes("UTF-8"));
        return signature.verify(signatureBytes);
    }
```
