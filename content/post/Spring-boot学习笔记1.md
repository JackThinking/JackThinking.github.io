---
title: Spring boot学习笔记
date: 2018-04-26 11:34:40
categories: 
    - Spring boot
tags:
    - Spring boot
---

跟着慕课网上的《2小时学会Spring Boot》学习，发现讲解比较简单，适合像我这样的初学者学习，记录一下每个章节学到的知识点，主要以学习springboot框架为主.
## Controller的使用
### 请求部分
1.@Controller:处理HTTP请求
2.@RestController:spring4之后新加的注解，原来返回json需要@RequestBody+@Controller
3.@RequestMapping:配置url映射

如果单独使用@Controller的话，需要配合模板使用，首先在pom.xml配置spring的thymeleaf，然后再resources目录下建立相应的templates文件夹，再在这个文件夹下写一个index.html，这种方法带来的性能损耗较大，实际开发中前后端分离，不会这样做的，后端提供接口和json格式的数据就行。

@RequestMapping可以将请求映射到多个url下，实际代码如下:
``` java
@RequestMapping(value = {"/hello"，"/hi"},method = RequestMethod.GET)
```
就使得下面的url内容一致。
``` java
http://127.0.0.1:8080/hello
http://127.0.0.1:8080/hi
```
还有就是可以在整体在前面加一个“hello”字段，然后下面再加“say”字段，如下：
``` java
@RestController
@RequestMapping("hello")
public class HelloController {

    @Autowired
    private GirlProperties girlProperties;

    @RequestMapping(value = "/say",method = RequestMethod.GET)
    public String say(){
        return girlProperties.getCupSize();
    }
}
```
### 参数部分
1.@PathVariable:获取url的数据
2.@RequestParam:获取请求参数的值
3.@GetMapping:组合注解

使用以下的代码:
``` java
@RequestMapping(value = "/say/{id}",method = RequestMethod.GET)
public String say(@PathVariable("id") Integer id){
    return "id" + id;
}
```
就可以获取到下面url中100的这个值。
``` java
http://127.0.0.1:8080/hello/say/100
```
使用以下的代码:
``` java
@RequestMapping(value = "/say",method = RequestMethod.GET)
public String say(@RequestParam(value = "id",required = false,defaultValue = "0") Integer myid){
    return "id" + myid;
 }
```
就可以获取到下面url中100的这个值。
``` java
http://127.0.0.1:8080/hello/say/?id=100
```
而且由于@RequestParam设置的默认参数，就算是下面这样：
``` java
http://127.0.0.1:8080/hello/say/?id=
```
也能显示id0。
由于下面代码效果等价，但是第二种写法更加短小，所以推荐使用第二种写法。
``` java
@RequestMapping(value = "/say",method = RequestMethod.GET)
@GetMapping(value = "/say")
```
