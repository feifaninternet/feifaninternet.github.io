---------------------------
title: 微信支付集成 -- JAVA
---------------------------
date: 2018-03-22 14:04:21
tags: 微信支付

### 1.文档说明

  微信支付封装--DANKAL JAVAEE    
  用于JAVA后台微信支付相关功能的开发。

#### 版本说明   
· 1.0.0 -- 2017-3-10 
支付签名及统一下单接口的封装。

### 2.协议规范
#### 应用场景
使用该工具在微信支付后台生成预支付交易单，返回正确的预支付交易回话标识后再在APP或者小程序中调起微信支付。

#### 使用方法
1. 导入 jar 包。
2. 构建 PaymentPO 对象 
3. 调用 WxPayResponse.getPaySign() 获取调起微信支付所需参数

#### 详细说明
### 1.PaymentPO 对象

    
    public class PaymentPO {

        /**
         * 交易金额 * 必须
         */
        private String total_fee;
    
        /**
         * appId * 必须
         */
        private String appid;
    
        /**
         * 商户号 * 必须
         */
        private String mch_id;
    
        /**
         * 商户密匙 * 必须
         */
        private String merchant_pay_key;
    
        /**
         * 用户 openId * 必须
         */
        private String openid;
    
        /**
         * 终端 ip (用户端 ip,请注意格式,格式不对也会异常) * 必须
         */
        private String spbill_create_ip;
    
        /**
         * 微信回调地址，用于通知微信支付结果 * 必须
         */
        private String notify_url;
    
        /**
         * 随机字符串 (若不传则默认使用封装随机字符串生成规则)
         */
        private String nonce_str;
    
        /**
         * 签名,内部生产(MD5方式)
         */
        private String sign;
    
        /**
         * 商品描述 (最好传,不传也有默认值)
         */
        private String body;
    
        /**
         * 订单号 (若不传则默认使用封装订单号生成规则)
         */
        private String out_trade_no;
    
        /**
         * 订单优惠标记 使用代金券或者立减优惠功能时需要的参数
         */
        private String goods_tag;
    
        /**
         * 交易类型 若不传默认为小程序支付(MINI: JSAPI ; APP: APP)
         */
        private String trade_type;
    
        /**
         * 交易开始时间
         */
        private String time_start;
    
        /**
         * 交易结束时间
         */
        private String time_expire;
    }
    
#### 构建示例：

    PaymentPO paymentPO = new PaymentPO();
    paymentPO.setAppid("wx65fbfa71fbf0b639");
    paymentPO.setMch_id("1498814312");
    paymentPO.setMerchant_pay_key("vnYncWrMQsGHTVvj8ZR76B1d8T32oQZK");
    paymentPO.setNotify_url("https://api-czh.dankal.cn/v1/recharge/wx_notify");
    paymentPO.setTotal_fee("100");
    paymentPO.setSpbill_create_ip("120.78.10.31");
    paymentPO.setOpenid("odyO05BBjHEqIrPu1R5k9WbNwaC4");
    
### 2.WxPayResponse 

  内部对 PaymentPO 对象的参数进行验证,必须的参数不能缺少，否则会视为异常，若参数正确，会请求微信支付统一下单接口，然后再次签名，返回调用微信支付所需参数。 
  getPaySign()：此方法用于获取调起微信支付所需参数。   
   
  参数说明：

小程序调起微信支付 api 所需参数：   

| 参数名称        | 参数说明           | 备注  |
| --------------- |:------------------:| -----:|
| nonceStr        | 随机字符串         | 不大于 32 位 |
| package         | 数据包             | prepayid=wx2017033010242291fcfe0db70013231072 统一下单接口返回的 prepayid 参数值|
| timeStamp       | 当前时间的时间戳   | 单位 : 秒 |
| paySign         | 签名               | 请求统一下单接口后,返回给前端需要再次签名 参与签名的参数 appId; nonceStr; package; signType; timeStamp; key|

&nbsp;&nbsp;详细参数说明及签名算法 : [WeChat paid official API](https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=7_7&index=5)   
  
返回示例：

#### success

    {
        timeStamp=1521442735, 
        paySign=2476BEEECA2A991D7A41257B0BD26E9C, 
        nonceStr=dd03919fa71742d282b5e12da59c25ce, 
        package=prepay_id=wx201803191458535042d396b30400714725
    }     
#### fail 
- 微信返回错误:


    {
        error = invalid spbill_create_ip
    }
- 参数错误:


    {
        error=[SpbillCreateIp is missing, TotalFee is missing]
    }    
    
### 3.示例代码

    public class TestWxPayResponse {
        public static void main(String[] args) {

            //构建 PaymentPO 对象,必须参数的赋值
            PaymentPO paymentPO = new PaymentPO();
            paymentPO.setAppid("wx65fbfa71fbf0b639");
            paymentPO.setMch_id("1498814312");
            paymentPO.setMerchant_pay_key("vnYncWrMQsGHTVvj8ZR76B1d8T32oQZK");
            paymentPO.setNotify_url("https://api-czh.dankal.cn/v1/recharge/wx_notify");
            paymentPO.setTotal_fee("100");
            paymentPO.setSpbill_create_ip("120.78.10.31");
            paymentPO.setOpenid("odyO05BBjHEqIrPu1R5k9WbNwaC4");
    
            try {
                //获取微信支付所需参数
                WxPayResponse wxPayResponse = new WxPayResponse(paymentPO);
                Map<String, Object> paySign = wxPayResponse.getPaySign();
                System.out.println(paySign);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    
### 扩展