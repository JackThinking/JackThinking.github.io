---
title: Spring boot学习笔记3
date: 2018-05-03 09:30:43
categories: 
    - Spring boot
tags:
    - Spring boot
---
之前按照spring boot的视频依葫芦画瓢，虽然知道了大体框架，但是很多细节方面的问题都被带过了，现在回顾一下。

## 框架
整个工程可以分为4个模块
1.java主程序
2.test单元测试模块
3.资源模块（.yml用于配置数据库)
4.maven的pom.xml模块（用于添加依赖）

## 数据库操作
这个工程用到的是spring boot的jpa功能，在实际的代码中不出现sql语句，而是采用传递变量的方式。
需要自定义一个接口文件（extends JpaRepository），在这个文件中代码很少，有许多默认的数据库操作函数，当然也可以自定义自己的操作函数：
```java
package com.citrix.girl.repository;

import com.citrix.girl.domain.Girl;
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.List;

public interface GirlResponsitory extends JpaRepository<Girl,Integer>{
//    通过年龄来查询
    public List<Girl> findByAge(Integer age);
}
```
### 数据库的对象
本次项目中的数据库对象是girl的属性，将其写成girl的类，定义里面的参数（与数据库中对于）有id,cupsize,age,money,然后再类里面写好必要的get set方法，再写一个tostring方法便于打印带控制台。
```java
package com.citrix.girl.domain;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;

//注解表示类对于数据库中的表
@Entity
public class Girl {
    
    @Id//表示表中的id
    @GeneratedValue//表示id是默认升序的
    private Integer id;
    private String cupSize;
    @Min(value = 18,message = "未成年少女禁止入门")//对age的最小值进行限定
    private Integer age;

    @NotNull(message = "金额必填")//对金额的有无进行限定
    private Double money;

    public Girl() {
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getCupSize() {
        return cupSize;
    }

    public void setCupSize(String cupSize) {
        this.cupSize = cupSize;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Double getMoney() {
        return money;
    }

    public void setMoney(Double money) {
        this.money = money;
    }
    @Override//因为重写了父类的方法
    public String toString() {
        return "Girl{" +
                "id=" + id +
                ", cupSize='" + cupSize + '\'' +
                ", age=" + age +
                ", money=" + money +
                '}';
    }
}
```
## 异常管理
首先，传统的异常只能接受string选项，功能受限，我们先定义自己的异常函数，主要是加入了异常代码：
```java
package com.citrix.girl.exception;
import com.citrix.girl.enums.ResultEnum;
//继承的是runtime的exception因为spring不会对一般的exception进行回滚
public class GirlException extends RuntimeException{
    private Integer code;
    public GirlException(ResultEnum resultEnum) {
        super(resultEnum.getMsg());//super是指父类的
        this.code = resultEnum.getCode();
    }
    public Integer getCode() {
        return code;
    }
    public void setCode(Integer code) {
        this.code = code;
    }
}
```
这里我们对异常的种类进行了枚举限定，因为这样会显得比较有有序：
```java
package com.citrix.girl.enums;
/**
 * Created by Citrix on 2018/4/29.
 */
//作用是限制自定义异常的错误码随便使用
public enum ResultEnum {
    UNKONW_ERROR(-1,"未知错误"),
    SUCCESS(0,"成功"),
    PRIMARY_SCHOOL(100,"还在上小学"),
    MIDDLE_SCHOOL(101,"还在上初中"),
    ;//枚举的显示表现
    private Integer code;
    private String msg;
    //枚举的构造函数
    ResultEnum(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }
    //基本的getset方法
    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```
做好异常的自定义之后，开始设计异常处理函数：
```java
package com.citrix.girl.handle;

import com.citrix.girl.domain.Girl;
import com.citrix.girl.domain.Result;
import com.citrix.girl.exception.GirlException;
import com.citrix.girl.util.ResultUtil;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
@ControllerAdvice
public class ExceptionHandle {

    private final static org.slf4j.Logger logger = LoggerFactory.getLogger(ExceptionHandle.class);
    @ExceptionHandler(value = Exception.class)//异常处理注解
    @ResponseBody//
    //捕获异常的方法
    public Result handle(Exception e) {
        //如果这个异常是在我自定义异常范围内的
        if (e instanceof GirlException) {
            GirlException girlException = (GirlException) e;
            return ResultUtil.error(girlException.getCode(), girlException.getMessage());
        } else {
            //显示未知错误异常
            logger.error("【系统异常】{}",e);
            return ResultUtil.error(-1, "未知错误");
        }
    }
}
```
最后，为了对前端数据的统一性，将正确信息和异常信息进行统一格式的处理：
```java
package com.citrix.girl.util;

import javax.xml.transform.Result;

public class ResultUtil {
    //正确操作的话
    public static com.citrix.girl.domain.Result success(Object object){
        com.citrix.girl.domain.Result result = new com.citrix.girl.domain.Result();
        result.setCode(0);
        result.setMsg("成功");
        result.setData(object);
        return result;

    }
    public static com.citrix.girl.domain.Result success(){
        return success(null);

    }
    //出错的话，将错误信息的msg进行返回
    public static com.citrix.girl.domain.Result error(Integer code,String msg){
        com.citrix.girl.domain.Result result = new com.citrix.girl.domain.Result();
        result.setCode(code);
        result.setMsg(msg);
        return result;
    }
}
```
其中的result也是对于返回数据进行了类的封装，为的是对前端来说更加统一：
```java
package com.citrix.girl.domain;
//这边的<T>还是不是很理解，实际使用的时候感觉T是一个类 
public class Result<T> {
    //错误码
    private Integer code;
    //提示信息
    private String msg;
    //具体的内容
    private T data;

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }
}
```

## 日志
```java
package com.citrix.girl.aspect;


import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.*;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import javax.servlet.ServletRequest;
import javax.servlet.http.HttpServletRequest;

@Aspect
@Component
public class HttpAspect {
    private final static org.slf4j.Logger logger = LoggerFactory.getLogger(HttpAspect.class);
    //这边是切面操作，是自定义的，还不是很懂，大概是执行GirlController时会监测
    @Pointcut("execution(public * com.citrix.girl.controller.GirlController.*(..))")
    public void log(){

    }
    //注解含义是在接受请求后进行处理之前，进行日志记录，"log()"是指之前与切面注解绑定的函数，这样就省得每次写，也是切面的好处
    @Before("log()")
    public void doBefore(JoinPoint joinPoint){
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        //url
        logger.info("url={}",request.getRequestURL());

        //method
        logger.info("method={}",request.getMethod());

        //ip
        logger.info("ip={}",request.getRemoteAddr());

        //类方法
        logger.info("class_method={}",joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName());

        //参数
        logger.info("args={}",joinPoint.getArgs());

    }
    //注解含义是在接受请求后进行处理之后，进行日志记录
    @After("log()")
    public void doAfter(){
        logger.info("22222222");

    }
    //注解含义是在返回给前端之后进行的操作
    @AfterReturning(returning = "object",pointcut = "log()")
    public void doAfterReturning(Object object){
        logger.info("response={}",object.toString());
    }
}
```
