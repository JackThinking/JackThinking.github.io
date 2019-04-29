---
title: Spring boot学习笔记4
date: 2018-05-03 20:07:36
categories: 
    - Spring boot
tags:
    - Spring boot
---
spring中jpa操作createTime对应于数据库的中的create_time,这个是固定的，jpa操作的时候会自动进行名字的转化，不能写错，负责就会造成数据库操作出错。

## getset方法太麻烦怎么办

1.在pom.xml添加lombok
2.idea添加插件lombok
3.在DAO类中添加注解@Data就可以取消getset以及tostring之类的方法了

## 单元测试
1.注意要使用Assert断言来确认数据的正确性。
2.@Transactional注解可以报使得测试的数据不会对数据库造成影响（只是测试一下嘛）
3.遇到了这样的语法，应该是数组的初始化吧
```java
List<Integer> list = Arrays.asList(2,3,4);
```

## 梳理一下流程

第一个数据的设计顺序是，先设计DAO（数据库对象），然后设计Service模块（注意接口与接口实现的分离），最后设计controller模块，其中还有很重要的是一定要先进行单元测试。

## Service服务写成接口（并且实现接口与接口实现分离）
拿CategoryService举个栗子🌰:
```java
package com.imooc.sell.service;

import com.imooc.sell.dataobject.ProductCategory;

import java.util.List;

/**
 * 类目接口
 * Created by Citrix on 2018/5/3.
 */
public interface CategoryService {

    ProductCategory findOne(Integer categoryId);//id查找一个

    List<ProductCategory> findAll();//查找全部

    List<ProductCategory> findByCategoryTypeIn(List<Integer> categoryTypeList);//根据商品种类找

    ProductCategory save(ProductCategory productCategory);//保存
}
```
具体的实现虽然不麻烦，只是重载之前ProductCategoryRepository的Jpa函数，但是还是这样比较清晰：
```java
package com.imooc.sell.service.impl;

import com.imooc.sell.dataobject.ProductCategory;
import com.imooc.sell.repository.ProductCategoryRepository;
import com.imooc.sell.service.CategoryService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * 类目接口具体实现
 * Created by Citrix on 2018/5/3.
 */
@Service
public class CategoryServiceImpl implements CategoryService{

    @Autowired
    private ProductCategoryRepository repository;

    @Override
    public ProductCategory findOne(Integer categoryId) {
        return repository.findById(categoryId).get();//这边本来是自带的findOne函数，不过在spring更新之后变成了findById而且还得加get()函数
    }

    @Override
    public List<ProductCategory> findAll() {
        return repository.findAll();
    }

    @Override
    public List<ProductCategory> findByCategoryTypeIn(List<Integer> categoryTypeList) {
        return repository.findByCategoryTypeIn(categoryTypeList);
    }

    @Override
    public ProductCategory save(ProductCategory productCategory) {
        return repository.save(productCategory);
    }
}
```
## 小技巧

1.cmd+shift+enter在IDEA中可以直接跳转到下一行，方便coding
2.Mac上的ctrl+N（我改变按键映射）可以快速使用IDEA提供的getset之类的自动生成函数

## 小问题

1.第一次遇到用BigDecimal类型的变量，作用是精确计算，具体使用方式暂且留着
2.在单元测试ProductServiceImplTest遇到了Page<ProductInfo>以及PageRequest的类，先留着
3.项目之后打成JAR包部署的时候会自带项目名称，在.yml文件中添加
```yml
server:
  servlet:
    context-path: /sell
```
4.遇到了lambda表达式，之后学习Java8语法的时候格外注意
5.用到了beanutil的方法，但不是很懂
```java
BeanUtils.copyProperties(productInfo,productInfoVO);//用到了bean的方法
productInfoVOList.add(productInfoVO);
```

## 又一次使用枚举
为了程序的清晰性和可维护性，将程序中使用的一些状态码写成枚举的类型：
```java
package com.imooc.sell.enums;

import lombok.Getter;

/**
 * Created by Citrix on 2018/5/4.
 */
@Getter//也是lombak的好东西
public enum ProductStatusEnum {
    //枚举数据类型
    UP(0,"在架"),
    DOWN(1,"下架")
    ;
    //变量
    private Integer code;
    private String message;
    //构造函数
    ProductStatusEnum(Integer code, String message) {
        this.code = code;
        this.message = message;
    }
}
```
## 回顾

@RestController是@ResponseBody和@Controller的组合注解，目的是返回json格式

## Json格式设计
```js
GET /sell/buyer/product/list
```

参数

```js
无
```

返回

```js
{
    "code": 0,
    "msg": "成功",
    "data": [
        {
            "name": "热榜",
            "type": 1,
            "foods": [
                {
                    "id": "123456",
                    "name": "皮蛋粥",
                    "price": 1.2,
                    "description": "好吃的皮蛋粥",
                    "icon": "http://xxx.com",
                }
            ]
        },
        {
            "name": "好吃的",
            "type": 2,
            "foods": [
                {
                    "id": "123457",
                    "name": "慕斯蛋糕",
                    "price": 10.9,
                    "description": "美味爽口",
                    "icon": "http://xxx.com",
                }
            ]
        }
    ]
}
```
比较复杂，在程序中分成了三层类实现（ResultVO+ProductVO+ProductInfoVO）：
```java
package com.imooc.sell.VO;

import lombok.Data;

/**
 * Created by Citrix on 2018/5/4.
 */
@Data
public class ResultVO<T> {

    /*错误码*/
    private Integer code;
    /*提示信息*/
    private String msg;
    /*具体内容*/
    private T data;
}
```
```java
package com.imooc.sell.VO;

import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.Data;

import java.util.List;

/**
 * 商品（包含类目）
 * Created by Citrix on 2018/5/4.
 */
@Data
public class ProductVO {

    @JsonProperty("name")
    private String categoryName;
    @JsonProperty("type")
    private Integer categoryType;
    @JsonProperty("foods")
    private List<ProductInfoVO> productVOList;
}
```
```java
package com.imooc.sell.VO;

import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.Data;

import java.math.BigDecimal;

/**
 * 商品详情
 * Created by Citrix on 2018/5/4.
 */
@Data
public class ProductInfoVO {

    @JsonProperty("id")
    private String productId;

    @JsonProperty("name")
    private String productName;

    @JsonProperty("price")
    private BigDecimal productPrice;

    @JsonProperty("description")
    private String productDescription;

    @JsonProperty("icon")
    private String productIcon;
}
```
其中的@JsonProperty("XXX")注解作用是，后端的变量和到时候传到前端的变量名字不一样，因为如果全按照之前的json格式在定义本地变量，必然造成变量名重复和混乱，还有一点，可以发现传回带前端的数据少于后端的数据，实际情况大多数也是，有些数据比如（库存）不会轻易的让别人知道。

## 返回数据统一化

为了避免在返回给前端的过程中重复写（定义变量,set一些，save一些，return之类），将其写成一个类，并且写好三个函数（带参success，无参success，带参error）
```java
package com.imooc.sell.utils;

import com.imooc.sell.VO.ResultVO;

/**
 * Created by Citrix on 2018/5/4.
 */
public class ResultVOUtil {
    public static ResultVO success(Object object){
        ResultVO resultVO = new ResultVO();
        resultVO.setData(object);
        resultVO.setCode(0);
        resultVO.setMsg("成功");
        return resultVO;
    }

    public static ResultVO success(){
        return success(null);
    }

    public static ResultVO error(Integer code, String msg){
        ResultVO resultVO = new ResultVO();
        resultVO.setCode(code);
        resultVO.setMsg(msg);
        return resultVO;
    }
}
```
