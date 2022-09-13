+++
author = "H. Wang"
title = "Long '==' & 'equals()'"
date = "2022-09-13"
description = "Long值的比较判断相等问题"
tags = [
    "Java"
]
categories = [
    "Java"
]

image = ""
+++

## Long值的比较判断相等问题

业务场景为数据库中主键id字段类型为`bigint`（因为雪花算法自增），后端代码使用对应的数据类型是`Long`。

```java
public DTree(Map<String, Object> node) {
    this.id = (Long) node.get("id");
    this.label = (String) node.get("label");
    this.parentId = (Long) node.get("parent_id");
    this.children = null;
}
```

实际上，MySQL中有符号类型 `Bigint(20)`的取值范围为-9223372036854775808~9223372036854775807，与Java.lang.Long的取值范围完全一致，mybatis/mybatis-plus会将 Bigint(20)映射为Long类型。
——[CSDN 梦 * 蝶](https://blog.csdn.net/LZ15932161597/article/details/110248316)

出现问题的情况为：

当主键id位数大于3时，后端组树的业务代码将所有数据都作为根节点，前端显示为非“树”状列表，即全为父节点。

排错后总结问题为id的判断出错，即`==`未能判断出两个id相等。类似的问题有 [CSDN 火火火FF
](https://blog.csdn.net/weixin_52163830/article/details/125930121)  和我的这个问题非常相似。

其实排错过程中遇到超三位数就不能判断出结果就比较容易猜到Long型判断问题。

问题比较有趣、典型，有必要记录一下。如
https://blog.csdn.net/cherry7005/article/details/112392459 中说明。

> - long是基本数据类型，判断是否相等时使用==，即可判断值是否相等。（基本数据类型没有equals()方法）
> - Long是引用数据类型，当其数值在[-128,127]之间时，能用==判断是否相等，亦可用 >、< 比较大小。

java.lang.Long.java中：

```java
private static class LongCache {
    private LongCache(){}

    static final Long cache[] = new Long[-(-128) + 127 + 1];

    static {
        for(int i = 0; i < cache.length; i++)
            cache[i] = new Long(i - 128);
    }
}
```

LongCache会预先缓存 [-128,127] 范围内的数，通过缓存频繁请求的值代来更好的空间和时间性能，当数据超出此范围，则new一个Long对象；

“==”是比较的地址，超出此范围的数据地址不一致，所以范围内的比较是true，范围外的数据是false。

另外，Java对于每一种基本数据类型都会创建一个缓存池。基本数据类型对应的缓存池如下：

> - boolean values true and false
> - all byte values
> - short values between -128 and 127
> - int values between -128 and 127
> - char in the range \u0000 to \u007F

使用对应的包装类型时，如果值范围在缓存池范围内，会直接引用缓存池中的对象，否则会创建新的对象。

`==`判断的是地址，故在缓存池范围内的判断为`true`，而超出范围为`false`。

所以对于包装类型的数值判断可以拆箱之后进行判断或使用equals().
