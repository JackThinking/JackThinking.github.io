---
title: Spring boot学习笔记8
date: 2018-05-13 10:02:56
categories: 
    - Spring boot
tags:
    - Spring boot
---


## 小知识
1.由于在pc端上修改了host文件，可以将网址sell.com直接解析为127.0.0.1，但是在手机上不能改host，如何访问到pc上的开发环境呢？可以用代理的方式，使用charles软件，手机连接到和pc同一个网段后使用代理就行。（前提是开启内网穿透，让外网可以访问到本机）
2.在paycontroller里面返回的是网页而不是json字符串，所以使用注解@Controller而不是@RestController
3.使用外部的资源的话，在maven的依赖里面添加即可，自动下载，还可以指明版本号
4.微信支付有一个异步通知的设置，要设置异步通知的网址接受并处理（TODO）
5.支付端的代码推荐写到后端，可以重复利用
6.想要实现后端生成动态的html文本的话，需要动态输入参数，使用Freemarker,java文件返回ModelandView类就行
```java
public ModelAndView create(@RequestParam("orderId") String orderId,
                       @RequestParam("returnUrl") String returnUrl,
                               Map<String, Object> map
    ){
        //1.查询订单
        OrderDTO orderDTO = orderService.findOne(orderId);
        if(orderDTO == null){
            throw new SellException(ResultEnum.ORDER_NOT_EXIST);

        }
        //2.发起支付
        PayResponse payResponse = payService.create(orderDTO);

        map.put("payResponse", payResponse);
        map.put("returnUrl", returnUrl);

        return new ModelAndView("pay/create",map);
    }
```
相应的create.ftl文件内容为：
```html
<script>
    function onBridgeReady(){
        WeixinJSBridge.invoke(
                'getBrandWCPayRequest', {
                    "appId":"${payResponse.appId}",     //公众号名称，由商户传入
                    "timeStamp":"${payResponse.timeStamp}",         //时间戳，自1970年以来的秒数
                    "nonceStr":"${payResponse.nonceStr}", //随机串
                    "package":"${payResponse.packAge}",
                    "signType":"MD5",         //微信签名方式：
                    "paySign":"${payResponse.paySign}" //微信签名
                },
                function(res){
//                    if(res.err_msg == "get_brand_wcpay_request:ok" ) {}     // 使用以上方式判断前端返回,微信团队郑重提示：res.err_msg将在用户支付成功后返回    ok，但并不保证它绝对可靠。
                    location.href = "${returnUrl}";
                }
        );
    }
    if (typeof WeixinJSBridge == "undefined"){
        if( document.addEventListener ){
            document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false);
        }else if (document.attachEvent){
            document.attachEvent('WeixinJSBridgeReady', onBridgeReady);
            document.attachEvent('onWeixinJSBridgeReady', onBridgeReady);
        }
    }else{
        onBridgeReady();
    }
</script>
```
7.在对账的时候，可以出现金额由于数据类型不同出错的情况，为此特意写了一个工具类MathUtil（判断精度到小数点后面两位）：
```java
package com.imooc.sell.utils;

/**
 * Created by Citrix on 2018/5/10.
 */
public class MathUtil {

    private static final Double Money_Range = 0.01;

    public static Boolean equals(Double num1, Double num2) {
        Double result = Math.abs(num1 - num2);
        if (result < Money_Range) {
            return true;
        }
        return false;
    }
}
```
8.处理微信发给来的异步通知信息，处理成功微信端就不发了：
```java
    /**
     * 异步回调
     */
    @PostMapping("/notify")
    public ModelAndView notify(@RequestBody String notifyData) {

        payService.notify(notifyData);

        //返回给微信处理结果
        return new ModelAndView("pay/success");

    }
```
9.在原来的基础上，增加退款逻辑，在OrderServiceImpl里面写cancel函数，paid函数，finish函数（finish和paid的区别再看看）
10.数据库中存的是上架下架等的状态码，而在网页端的时候的时候需要是msg格式的，所以需要写一个根据枚举里面code找到msg的方法EnumUtil：
```java
package com.imooc.sell.utils;

import com.imooc.sell.enums.CodeEnum;

/**
 * Created by Citrix on 2018/5/11.
 */
public class EnumUtil {
    public static <T extends CodeEnum> T getByCode(Integer code, Class<T> enumClass) {
        for (T each: enumClass.getEnumConstants()) {
            if (code.equals(each.getCode())) {
                return each;
            }
        }
        return null;
    }
}
```
11.有时候需要某些数据返回到前端json的时候自动省略（这些个函数主要用于生成前端页面），则可以加注解@JsonIgnore
## 小技巧
1.为了控制台输出的数据格式直观，写了一个工具类JsonUtil，作用就是将数据变成json格式打印
2.使用网站ibootstrap生成简单的前端页面代码

## 小问题
1.虽然获取openid可以用测试账号，但是支付的开发必须使用服务号的功能，奈何太贵，这不过就不测试开发了
2.微信支付有个预支付参数（prepayid）之后如果有需要再看看
3.Java语法里面Map的使用
4.微信退款的操作需要额外的证书，这边先不尝试
5.微信的退款通知也放放
