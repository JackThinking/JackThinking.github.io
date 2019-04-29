---
title: Spring boot学习笔记7
date: 2018-05-08 20:07:37
categories: 
    - Spring boot
tags:
    - Spring boot
---
今天暂停一下跟着视频学习的进程，好好回顾一下程序的逻辑。

## 获取yml配置文件里的变量
比如我在yml配置的微信关公众号的信息如下：
```js
wechat:
  mpAppid: wxf98e335974fbfa76
  mpAppSecret: 7aee54b6a921a28f8f53c03590d05b20
```
在
```java
package com.imooc.sell.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.Map;

/**
 * Created by Citrix on 2018/5/7.
 */
@Data
@Component//泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注
@ConfigurationProperties(prefix = "wechat")//prefix后面跟着 的就是在yml文件里配置的字段名
public class WechatAccountConfig {

    private String mpAppId;

    private String mpAppSecret;

    /**
     * 商户号
     */
    private String mchId;

    /**
     * 商户密钥
     */
    private String mchKey;

    /**
     * 商户证书路径
     */
    private String keyPath;
}
```
## 主要注解理解
1.@Service用于标注业务层组件

2.@Controller用于标注控制层组件（如struts中的action）

3.@Repository用于标注数据访问组件，即DAO组件(我的学习例子DAO里没有，出现在配置文件里)

4.@Component泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。

## OrderService的create逻辑梳理
这部分在学习的时候就觉得业务逻辑复杂度较高，现在来一步步理解。
首先对于数据库的字段设计相应的类DAO,但是其业务逻辑导致DAO不能完全胜任工作（order的list之类，就不细讲了），故再设计DTO
1.首先判断下单的产品还有没有（productInfo的ProductId）
2.都在的话，找到单价，并且求订单总价(用到了lambda，productInfo的ProductPrice)
3.将订单的id和订单详情的id记下来(都是随机数加时间戳)
4.在订单详情中将产品信息的内容复制过去（BeanUtils.copyProperties(productInfo,orderDetail)）
5.在订单中写入id，orderDTO的内容（买家的信息），订单总价，订单的支付状态和下单状态
6.订单（orderMaster）和订单详情（orderDetail）存到数据库中
7.扣库存，将调用productService进行decreaseStock操作

## 关于Page和List
首先分析下面这段Page有关的代码：
```java
public Page<OrderDTO> findList(String buyerOpenid, Pageable pageable) {
    Page<OrderMaster> orderMasterPage = orderMasterRepository.findByBuyerOpenid(buyerOpenid, pageable);

    List<OrderDTO> orderDTOList = OrderMaster2OrderDTOConverter.convert(orderMasterPage.getContent());
    //???
    return new PageImpl<>(orderDTOList, pageable, orderMasterPage.getTotalElements());
    }
```
## 获取用户的OpenID

## 重定向原理
