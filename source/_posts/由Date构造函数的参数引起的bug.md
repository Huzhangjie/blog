---
title: 由Date构造函数的参数引起的bug
date: 2021-11-27 21:00:00
subtitle:
categories: 基础
tags: 
  - javascript
cover: /images/b62b444a880511ebb6edd017c2d2eca2.jpg
---

## 起源
事情的起源是在修改bug，调试同事的代码时。发现明明是两个同样的日期时间`date1: 2021-02-08`, `date2: 2021-02-8`。但在比较两个日期时，却是`date1`在`date2`之后。这引起了我的困惑


细细排查之后才发现，原来上述两个字符串`new Date()`之后，实际的时间戳是不相等的
![image.png](https://i.loli.net/2021/11/27/NzRCL7FMxlwZjJ5.png)

查阅[MDN Date](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date) 文档后才发现，小小的`Date构造函数`，也有大大的学问。

## Date构造函数
### 语法
> new Date();
> new Date(value);
> new Date(dateString);
> new Date(year, monthIndex [, day [, hours [, minutes [, seconds [, milliseconds]]]]]);

`Date()`构造函数有四种基本形式
1.**没有参数**: 如果没有提供参数，那么新创建的Date对象表示实例化时刻的日期和时间
2.**Unix时间戳**: 
  - `value`: 一个 Unix 时间戳，它是一个整数值，表示自1970年1月1日00:00:00 UTC（the Unix epoch）以来的毫秒数。
3.**时间戳字符串**:
  - `dateString`: 表示日期的字符串值。该字符串应该能被 [Date.parse()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date/parse) 正确方法识别（即符合 [IETF-compliant RFC 2822 timestamps](https://datatracker.ietf.org/doc/html/rfc2822#page-14) 或 [version of ISO8601](https://262.ecma-international.org/5.1/#sec-15.9.1.15)）。
4.**提供日期与时间的每一个成员**:
  - `year`: 年份。0到99会被映射至1900年至1999年，其它值代表实际年份。
  - `monthIndex`: 表示月份的整数值，从 0（1月）到 11（12月）。
  - `date 可选`: 表示一个月中的第几天的整数值，从1开始。默认值为1。
  - `hours 可选`: 表示一天中的小时数的整数值 (24小时制)。默认值为0（午夜）。
  - `minutes 可选`: 表示一个完整时间（如 01:10:00）中的分钟部分的整数值。默认值为0。
  - `seconds 可选`: 表示一个完整时间（如 01:10:00）中的秒部分的整数值。默认值为0。
  - `milliseconds 可选`: 表示一个完整时间的毫秒部分的整数值。默认值为0。


### 时间戳字符串
这篇文章，我们主要了解下第三种构造函数的使用方式。记录一下其注意事项
> **注意**: 由于浏览器之间的差异与不一致性，强烈不推荐使用Date构造函数来解析日期字符串 (或使用与其等价的Date.parse)。对 RFC 2822 格式的日期仅有约定俗成的支持。 对 ISO 8601 格式的支持中，仅有日期的串 (例如 "1970-01-01") 会被处理为 UTC 而不是本地时间，与其他格式的串的处理不同。

那么上面这一段是什么意思呢，我们先来两张图了解一下`RFC 2822 格式`和`ISO 8601 格式`。具体的以后再讲。
![RFC 2822](https://i.loli.net/2021/11/27/mpEWKSg8iUYXzt5.png)
![ISO 8601](https://i.loli.net/2021/11/27/zEjkHLrBQSWIy5G.png)

文章开头展示了`2021-02-08`和`2021-02-8`经过`Date`构造函数生成实例不同，让我们来看几个更详细的栗子:
```js
/**
 * 预期之外的表现
 * 生成的日期是UTC时间，非系统所在时区时间
 * 以下输出都是: Mon Feb 08 2021 08:00:00 GMT+0800 (中国标准时间)
 */
// ISO 8601标准格式 YYYY-MM-DDTHH:mm:ss.sssZ
new Date('2021-02-08T00:00:00.000Z')
new Date('2021-02-08')


/**
 * 预期之内的时间
 * 即系统所在时区的日期的 0时
 * 以下输出都是: Mon Feb 08 2021 00:00:00 GMT+0800 (中国标准时间)
 */
new Date('2021/02/08')
new Date('20210.02.08')
new Date('2021-02-8')
new Date('2021-2-08')
new Date('2021.2.08')
new Date('2021/2.08')
...

```
大家都知道，`new Date()`会默认生成系统（执行JavaScript的客户端电脑）所在时区的时间，而非标准`UTC`（格林威治）时间。*关于时区的详细介绍看这篇文章(待补充)*

可以发现，除了以`ISO 8601标准格式`和`yyyy-MM-dd`格式的字符串传入，实例返回是`UTC`时间外（可以看到输出的加了8个小时的）。其余的都是符合直观上的预期的（都是当前时区的0点）。

挖了两个小坑，时区的和两个时间格式的解读。待补充吧

