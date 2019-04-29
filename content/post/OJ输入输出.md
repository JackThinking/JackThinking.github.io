---
title: OJ输入输出
date: 2019-04-03 22:28:03
categories: 
    - 算法
tags:
    - OJ
---
说来真是生气，今天的华为笔试第一题就是不确定长度的输入，之前一直没有碰到过，一紧张被卡了。索性把常见的几种输入输出在这里总结一下，我自己用BufferedReader比较多（据说速度快，但是类型转换以及功能没有scanner全）。
# 单行输入
## 输入输出
```
Input: 
1 2 3 4
Output: 
1
[1,2,3,4]
```
## 代码
```java
package io;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Arrays;

/**
 * Created by Citrix on 2019-04-03.
 */
public class singleLine {
    public static void main(String[] args) throws IOException {
        BufferedReader input = new BufferedReader(new InputStreamReader(System.in));
        String str = input.readLine();
        String[] inputLine = str.split(" ");
        int m = Integer.parseInt(inputLine[0]);
        System.out.println(m);
        System.out.println(Arrays.toString(inputLine));
    }
}
```
# 第一行参数，第二行数组
## 输入输出
```
Input: 
3
1，2，3
Output: 
3
[1,2,3]
```
## 代码
```java
package io;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Arrays;

/**
 * Created by Citrix on 2019-04-03.
 */
public class multiLine {
    public static void main(String[] args) throws IOException {
        BufferedReader input = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(input.readLine());
        String[] inputString = input.readLine().split(" ");
        int inputInt[] = new int[inputString.length];
        for (int i = 0; i < inputString.length; i++) {
            inputInt[i] = Integer.parseInt(inputString[i]);
        }
        System.out.println(n);
        System.out.println(Arrays.toString(inputInt));
    }
}
```
# 确定行数读取
## 输入输出
```
Input: 
3 3
123
234
345
Output: 
[1,2,3][2,3,4][3,4,5]
```
## 代码
```java
package io;

import sun.security.krb5.internal.crypto.Aes128;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * Created by Citrix on 2019-04-03.
 */
public class sureLine {
    public static void main(String[] args) throws IOException {
        BufferedReader input = new BufferedReader(new InputStreamReader(System.in));
        String[] pars = input.readLine().split(" ");
        int m = Integer.parseInt(pars[0]);
        int n = Integer.parseInt(pars[1]);
        List<String[]> inputAllLine = new ArrayList<>();
        for (int i = 0; i < m; i++) {
            String[] inputLine = input.readLine().split("");
            inputAllLine.add(inputLine);
        }
        String outputAllLine = "";
        for (int i = 0; i < inputAllLine.size(); i++) {
            outputAllLine += Arrays.toString(inputAllLine.get(i));
        }
        System.out.println(outputAllLine);
    }
}
```

# 不确定行数读取

## 输入输出
```
Input: 
1
1,1,1
2,2,2
3,3,3
...
n,n,n
Output: 
[1,1,1][2,2,2][3,3,3]...[n,n,n]
```
## 代码
```java
package io;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * Created by Citrix on 2019-04-03.
 */
public class unsureLine {
    public static void main(String[] args) throws IOException {
        BufferedReader input = new BufferedReader(new InputStreamReader(System.in));
        int n = Integer.parseInt(input.readLine());
        List<String[]> inputData = new ArrayList<>();
        String lineData = input.readLine();
        while (!lineData.equals("")) {
            String[] strings = lineData.split(",");
            inputData.add(strings);
            lineData = input.readLine();
        }
        String outputData = "";
        for (int i = 0; i < inputData.size(); i++) {
            outputData += Arrays.toString(inputData.get(i));
        }
        System.out.println(outputData);
    }
}
```

