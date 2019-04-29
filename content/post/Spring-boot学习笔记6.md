---
title: Spring boot学习笔记6
date: 2018-05-07 09:34:29
categories: 
    - Spring boot
tags:
    - Spring boot
---
周末两天把最硬核的买家端DAO-Service(单元测试)-Controller部分完成了，干货非常多，但是也有很多业务逻辑方面的代码需要再好好理解。
## 小问题
1.在OrderServiceImpl有CollectionUtils.isEmpty(orderDetailList)还不是很理解其语法
2.PageIml，getContent()以及Page类
3.@Valid注解和BindingResult
4.HashMap以及map的使用
```java
Map<String, String> map = new HashMap<>();
map.put("orderId", createResult.getOrderId());
```
## 小技巧
1.cmd+.可以在IDEA里将括号收起来
2.IDEA注释写/TODO可以显示高亮和跳转，方便后期修改
3.在postman里面可以把json的数据放进去自动生成key-value字段
4.IDEA里面如果出现删除线说明方法被弃用了，点进去可以看到修改后的方法

## 小知识
1.由于经常写DAO->DTO,索性写一个转换函数converter：
```java
package com.imooc.sell.converter;

import com.imooc.sell.dataobject.OrderMaster;
import com.imooc.sell.dto.OrderDTO;
import org.springframework.beans.BeanUtils;

import java.util.List;
import java.util.stream.Collectors;

/**
 * Created by Citrix on 2018/5/6.
 */
public class OrderMaster2OrderDTOConverter {
    //单个参数的函数
    public static OrderDTO convert(OrderMaster orderMaster) {

        OrderDTO orderDTO = new OrderDTO();
        BeanUtils.copyProperties(orderMaster, orderDTO);
        return orderDTO;
    }
    //列表参数的一个函数
    public static List<OrderDTO> convert(List<OrderMaster> orderMasterList) {
        return orderMasterList.stream().map(e ->
                convert(e)
        ).collect(Collectors.toList());
    }
}
```
2.对于前端传过来的表单设立专门的Form类：
```java
package com.imooc.sell.form;

import lombok.Data;

import javax.validation.constraints.NotEmpty;

/**
 * 表单验证模块，涉及到前端过来的表单验证
 * Created by Citrix on 2018/5/6.
 */
@Data
public class OrderForm {

    /**
     * 买家姓名
     */
    @NotEmpty(message = "姓名必填")
    private String name;

    /**
     * 买家手机号
     */
    @NotEmpty(message = "手机号必填")
    private String phone;

    /**
     * 买家地址
     */
    @NotEmpty(message = "地址必填")
    private String address;

    /**
     * 买家微信openid
     */
    @NotEmpty(message = "openid必填")
    private String openid;

    /**
     * 这个是麻烦的地方
     * 购物车
     */
    @NotEmpty(message = "购物车不能为空")
    private String items;
}
```
3.在自定义异常中添加message字段，扩展异常处理的多样性
4.BeanUtil的方法需要两个对象的类内变量名字相同
5.使用了Google的Gson格式，在maven里面添加依赖，然后gson在项目中的应用主要在OrderForm2OrderDTOConverter里面（还待研究）:
```java
package com.imooc.sell.converter;

import com.google.gson.Gson;
import com.google.gson.reflect.TypeToken;
import com.imooc.sell.dataobject.OrderDetail;
import com.imooc.sell.dto.OrderDTO;
import com.imooc.sell.enums.ResultEnum;
import com.imooc.sell.exception.SellException;
import com.imooc.sell.form.OrderForm;
import lombok.extern.slf4j.Slf4j;

import java.util.ArrayList;
import java.util.List;

/**
 * Created by Citrix on 2018/5/6.
 */
@Slf4j
public class OrderForm2OrderDTOConverter {

    public static OrderDTO convert(OrderForm orderForm) {
        Gson gson = new Gson();
        OrderDTO orderDTO = new OrderDTO();

        orderDTO.setBuyerName(orderForm.getName());
        orderDTO.setBuyerPhone(orderForm.getPhone());
        orderDTO.setBuyerAddress(orderForm.getAddress());
        orderDTO.setBuyerOpenid(orderForm.getOpenid());

        List<OrderDetail> orderDetailList = new ArrayList<>();
        /*防止出错*/
        try {
            /*这边是难点*/
            orderDetailList = gson.fromJson(orderForm.getItems(),
                    new TypeToken<List<OrderDetail>>() {
                    }.getType());
        } catch (Exception e) {
            /*经常配合log使用*/
            log.error("【对象转换】错误, string={}", orderForm.getItems());
            throw new SellException(ResultEnum.PARAM_ERROR);
        }
        orderDTO.setOrderDetailList(orderDetailList);

        return orderDTO;
    }
}
```
6.如果想要做到返回字段为Null就不传的效果的话，全局可以在.yml配置文件里添加：
```js
jackson:
    default-property-inclusion: non_null
```
局部的话，在相应的类里面（比如OrderDTO）添加注解@JsonInclude(JsonInclude.Include.NON_NULL)
