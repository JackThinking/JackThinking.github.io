---
title: Spring boot学习笔记2
date: 2018-04-27 16:01:21
categories: 
    - Spring boot
tags:
    - Spring boot
---
慕课网上的《2小时学会Spring Boot》学习结束,后半段主要讲了spring通过jpa操作Mysql数据库的操作，用到了测试软件postman（亲测很好用，界面又好看），在mac端安装了MySQL数据库，使用mac端的Navicat Premium作为数据库的workbench（这个老师虽然讲得好，但是用的例子略显猥琐，我这边虽然照着他也用了这个例子，可是这并不是我的本意/(ㄒoㄒ)/~~）。

## spring-data-jpa
JPA全称Java Persistence API.JPA通过JDK 5.0注解或XML描述对象－关系表的映射关系，并将运行期的实体对象持久化到数据库中。
### 利用JPA创建MySQL数据库
pom.xml加入JPA和MySQL的依赖
``` java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```
### 配置JPA和数据库
``` java
spring:
  profiles:
    active: a
  datasource:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://127.0.0.1:3306/dbgirl
      username: root
      password: 你猜

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```
### 创建与数据表对应的实体类Person
这边就不贴代码了，我懒。
### 创建接口GirlResponsitory
接口继承自JpaRepository，在之后控制部分都要用到GirlResponsitory。其中findByAge函数是自定义的用来根据年龄找出女生数组的函数。

``` java
public interface GirlResponsitory extends JpaRepository<Girl,Integer>{
//    通过年龄来查询
    public List<Girl> findByAge(Integer age);

}
```
### 创建控制器GirlController
这个是核心的操作数据库逻辑的文件，具体不解释了，看代码如下：
``` java
package com.citrix.girl;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import javax.persistence.criteria.CriteriaBuilder;
import javax.sql.rowset.serial.SerialStruct;
import java.util.List;

@RestController
public class GirlController {

    @Autowired
    private GirlResponsitory girlResponsitory;
    @Autowired
    private GirlService girlService;
    /*
    * 查询女生列表
    * @return
    * */
    @GetMapping(value = "/girls")
    public List<Girl> girlList(){
        return girlResponsitory.findAll();
    }
    /*
    * 添加一个女生
    * */
    @PostMapping(value = "/girls")
    public Girl girlAdd(@RequestParam("cupSize") String cupSize,
                          @RequestParam("age") Integer age) {
        Girl girl = new Girl();
        girl.setCupSize(cupSize);
        girl.setAge(age);

        return girlResponsitory.save(girl);
    }
    /*
    * 查询一个女生
    * */
    @GetMapping(value = "/girls/{id}")
    public Girl girlFindOne(@PathVariable("id") Integer id){

        return girlResponsitory.findById(id).get();
    }
    /*
    * 更新一个女生
    * */
    @PutMapping(value = "/girls/{id}")
    public Girl girlUpdate(@PathVariable("id") Integer id,
                           @RequestParam("cupSize") String cupSize,
                           @RequestParam("age") Integer age
                           ){
        Girl girl = new Girl();
        girl.setId(id);
        girl.setCupSize(cupSize);
        girl.setAge(age);

        return girlResponsitory.save(girl);
    }
    /*
    * 删除一个女生
    * */
    @DeleteMapping(value = "/girls/{id}")
    public void girlDelete(@PathVariable("id") Integer id){
        girlResponsitory.deleteById(id);
    }
    /*
    * 根据年龄来查找女生列表
    * */
    @GetMapping(value = "/girls/age/{age}")
    public List<Girl> girlListByAge(@PathVariable("age") Integer age){
        return girlResponsitory.findByAge(age);
    }
    /*
    *添加两个女生，相应的处理在GirlService里面
    * */
    @PostMapping(value = "/girls/two")
    public void girlTwo(){
        girlService.insertTwo();
    }
}
```
### 创建服务GirlService
添加Service方法的事务注解@Transactional，该方法遇到错误即可自动回滚，代码如下：
``` java
package com.citrix.girl;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import javax.transaction.Transactional;

@Service
public class GirlService {
    @Autowired
    private GirlResponsitory girlResponsitory;
    
    @Transactional
    public void insertTwo(){
        Girl girlA = new Girl();
        girlA.setCupSize("A");
        girlA.setAge(18);
        girlResponsitory.save(girlA);

        Girl girlB = new Girl();
        girlB.setCupSize("B");
        girlB.setAge(19);
        girlResponsitory.save(girlB);
    }
}
```
其中@Transactional注解配合@Service可以使得，两条数据要嘛同时插入，要不同时不插入。
