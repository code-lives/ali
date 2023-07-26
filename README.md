<p align="center">
<a href="https://packagist.org/packages/code-lives/ali" target="_blank"><img src="https://img.shields.io/packagist/v/code-lives/ali?include_prereleases" alt="GitHub forks"></a>
<a href="https://packagist.org/packages/code-lives/ali" target="_blank"><img src="https://img.shields.io/github/stars/code-lives/ali?style=social" alt="GitHub forks"></a>
<a href="https://github.com/code-lives/ali/fork" target="_blank"><img src="https://img.shields.io/github/forks/code-lives/ali?style=social" alt="GitHub forks"></a>

</p>

|                         第三方                          | token | openid | 支付 | 回调 | 退款 | 订单查询 | 解密手机号 | 分账 | 模版消息 |
| :-----------------------------------------------------: | :---: | :----: | :--: | :--: | :--: | :------: | :--------: | :--: | :------: |
| [支付宝小程序](https://opendocs.alipay.com/mini/03l5wn) |   x   |   ✓    |  ✓   |  ✓   |  ✓   |    ✓     |     ✓      |  x   |    ✓     |

## 安装

```php
composer require code-lives/ali 1.0.0
```

### ⚠️ 注意

> 金额单位分 100=1 元

> 返回结果 array 由开发者自行判断

# 预下单

```php

//引入命名空间
use Applet\Assemble\Ali;

// 小程序下单
$pay= Ali::init($config)->set("订单号","金额","描述",'openid')->getParam();

```

# 支付宝小程序

> 使用密钥进行签名解密，没有使用证书签名解密。

> 订单查询、退款、参数设置可以设置其他，具体看文档。

> 返回值 看官方文档，每个返回值都不一样，自行判断，如 openid 返回[alipay_system_oauth_token_response] 退款返回[alipay_trade_create_response]

### Config

| 参数名字   | 类型   | 必须 | 说明                         |
| ---------- | ------ | ---- | ---------------------------- |
| appid      | string | 是   | 小程序 appid                 |
| secret     | string | 是   | 小程序 AES 用于手机号解密    |
| privateKey | string | 是   | 应用私钥（开发工具生成）     |
| publicKey  | string | 是   | 支付宝公钥（支付宝后台下载） |
| notify_url | string | 是   | 异步回调地址                 |

### Openid

> getOpenid 获取支付宝的用户 user_id 类似于微信的 openid

```php
$code="";
$data= Ali::init($config)->getOpenid($code);
//返回参数
$data = array(
    [alipay_system_oauth_token_response] => Array
        (
            [access_token] => 123
            [alipay_user_id] => 123
            [auth_start] => 2023-03-26 20:56:36
            [expires_in] => 1296000
            [re_expires_in] => 2592000
            [refresh_token] => auth
            [user_id] => 123
        )
    [sign] =>
    )
```

### 解密手机号

```php
$data= Ali::init($config)->decryptPhone($session_key, $iv, $encryptedData);
echo $phone['mobile'];
```

### 订单查询

```php
$data = Ali::init($config)->findOrder(['out_trade_no' => '1679838318']);
```

### 退款

| 参数名字      | 类型   | 必须 | 说明       |
| ------------- | ------ | ---- | ---------- |
| out_trade_no  | string | 是   | 平台订单号 |
| refund_amount | int    | 是   | 退款金额   |

```php
$orders = [
        'out_order_no' => $order['out_order_no'],
        'refund_amount' => $order['refund_amount'],
    ];
$data= Ali::init($config)->applyOrderRefund($order);
```

### 模版消息

模版消息设置比较麻烦。需要先到开发平台添加进入小程序进行产品绑定，在去商家平台设置[文档](https://opendocs.alipay.com/b/03ksho)

```php
$data = [
        'to_user_id' => '用户uid',
        'user_template_id' => '模版id',
        'page' => 'pages/index/index',
        'data' => json_encode([
            'keyword1' => ['value' => '1'],
            'keyword2' =>  ['value' => '2'],
            'keyword3' => ['value' => '3'],
        ]),
    ];
$data= Ali::init($config)->sendMsg($data,$token);
```

## 支付回调

```php
$pay = Ali::init($config);
$status = $pay->notifyCheck(); //验证
if ($status) {
    $order = $pay->getNotifyOrder(); //订单数据
    //$order['out_trade_no']//平台订单号
}
```
