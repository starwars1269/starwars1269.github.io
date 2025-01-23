## 1. PayPal后台创建产品及计划

### 正式环境创建订阅计划
访问以下地址创建正式环境的订阅计划：  
[https://www.paypal.com/merchantapps/appcenter/acceptpayments/subscriptions](https://www.paypal.com/merchantapps/appcenter/acceptpayments/subscriptions)

### 沙盒环境创建订阅计划
访问以下地址创建沙盒环境的订阅计划：  
[https://www.sandbox.paypal.com/billing/plans](https://www.sandbox.paypal.com/billing/plans)

进入页面后，可以根据流程创建产品及计划。需要注意的是，正式环境和沙盒环境的计划是分开的，沙盒环境的计划不会显示在正式环境中。

此外，也可以通过API方式创建产品及计划，具体方法见下文。

---

## 2. API创建产品及计划

### API地址
访问以下地址查看PayPal API文档：  
[https://developer.paypal.com/api/rest/](https://developer.paypal.com/api/rest/)

### 1. 生成Token
在PayPal账号中获取`clientId`和`Secret`，然后通过以下地址获取`access_token`：

- **沙盒环境**: `https://api.sandbox.paypal.com/v1/oauth2/token`  
- **正式环境**: `https://api.paypal.com/v1/oauth2/token`

### 2. 创建产品
使用以下API请求创建产品：

bash
curl -v -X POST https://api-m.sandbox.paypal.com/v1/catalogs/products \
-H "Content-Type: application/json" \
-H "Authorization: Bearer Access-Token" \
-H "PayPal-Request-Id: PRODUCT-18062020-001" \
-d '{
  "name": "Video Streaming Service",
  "description": "Video streaming service",
  "type": "SERVICE",
  "category": "SOFTWARE",
  "image_url": "https://example.com/streaming.jpg",
  "home_url": "https://example.com/home"
}'


参数说明：
- `name`: 产品名称
- `description`: 产品说明
- `type`: 产品类型（`PHYSICAL`、`DIGITAL`、`SERVICE`）
- `category`: 产品类别
- `image_url`: 产品Logo地址
- `home_url`: 产品主页地址

### 3. 创建计划
使用以下API请求创建订阅计划：

bash
curl -v -X POST https://api-m.sandbox.paypal.com/v1/billing/plans \
-H "Content-Type: application/json" \
-H "Authorization: Bearer Access-Token" \
-H "PayPal-Request-Id: PLAN-18062019-001" \
-d '{
  "product_id": "PROD-XXCD1234QWER65782",
  "name": "Video Streaming Service Plan",
  "description": "Video Streaming Service basic plan",
  "status": "ACTIVE",
  "billing_cycles": [
    {
      "frequency": {
        "interval_unit": "MONTH",
        "interval_count": 1
      },
      "tenure_type": "REGULAR",
      "sequence": 1,
      "total_cycles": 12,
      "pricing_scheme": {
        "fixed_price": {
          "value": "6",
          "currency_code": "USD"
        }
      }
    }
  ],
  "payment_preferences": {
    "auto_bill_outstanding": true,
    "setup_fee": {
      "value": "6",
      "currency_code": "USD"
    },
    "setup_fee_failure_action": "CONTINUE",
    "payment_failure_threshold": 3
  },
  "taxes": {
    "percentage": "0",
    "inclusive": false
  }
}'


参数说明：
- `product_id`: 产品ID
- `name`: 计划名称
- `description`: 计划说明
- `status`: 计划状态
- `billing_cycles`: 计费周期配置
  - `frequency`: 计费频率
    - `interval_unit`: 时间间隔（`DAY`、`WEEK`、`MONTH`、`YEAR`）
    - `interval_count`: 间隔数量
  - `tenure_type`: 计费周期类型（`REGULAR`或`TRIAL`）
  - `sequence`: 计费周期顺序
  - `total_cycles`: 总计费周期数
  - `pricing_scheme`: 定价方案
    - `fixed_price`: 固定金额
- `payment_preferences`: 付款偏好设置
  - `auto_bill_outstanding`: 是否自动计费未结金额
  - `setup_fee`: 设置费
  - `setup_fee_failure_action`: 设置费失败后的操作（`CONTINUE`或`CANCEL`）
  - `payment_failure_threshold`: 连续失败次数阈值
- `taxes`: 税费配置

### 4. 查询计划
使用以下API请求查询计划详情：

bash
curl -v -X GET https://api-m.sandbox.paypal.com/v1/billing/plans/P-5ML4271244454362WXNWU5NQ \
-H "Content-Type: application/json" \
-H "Authorization: Bearer Access-Token"


### 5. 创建订阅
使用以下API请求创建订阅：

bash
curl -v -X POST https://api-m.sandbox.paypal.com/v1/billing/subscriptions \
-H "Content-Type: application/json" \
-H "Authorization: Bearer <Access-Token>" \
-H "PayPal-Request-Id: SUBSCRIPTION-21092019-001" \
-d '{
  "plan_id": "P-5ML4271244454362WXNWU5NQ",
  "start_time": "2022-07-21T00:00:00Z",
  "quantity": "20",
  "shipping_amount": {
    "currency_code": "USD",
    "value": "10.00"
  },
  "application_context": {
    "brand_name": "walmart",
    "locale": "en-US",
    "shipping_preference": "SET_PROVIDED_ADDRESS",
    "user_action": "SUBSCRIBE_NOW",
    "payment_method": {
      "payer_selected": "PAYPAL",
      "payee_preferred": "IMMEDIATE_PAYMENT_REQUIRED"
    },
    "return_url": "https://example.com/returnUrl",
    "cancel_url": "https://example.com/cancelUrl"
  }
}'


参数说明：
- `plan_id`: 计划ID
- `start_time`: 第二次扣款时间（ISO8601格式）
- `quantity`: 产品数量
- `shipping_amount`: 运费金额
- `application_context`: 应用上下文配置
  - `brand_name`: 品牌名称
  - `locale`: 语言
  - `shipping_preference`: 配送偏好
  - `user_action`: 用户操作（`CONTINUE`或`SUBSCRIBE_NOW`）
  - `payment_method`: 付款方式
  - `return_url`: 支付成功跳转地址
  - `cancel_url`: 支付取消跳转地址

根据返回结果，选择`links`下第一个`href`链接完成支付订阅。

### 6. 查询订阅详情
使用以下API请求查询订阅详情：

bash
curl -v -X GET https://api-m.sandbox.paypal.com/v1/billing/subscriptions/I-BW452GLLEP1G \
-H "Content-Type: application/json" \
-H "Authorization: Bearer Access-Token"


---

## 广告推荐

👉 [WildCard | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)