# Java基础易错点

## 1.基础

### 1.1正确使用equals方法

1.Object的equals方法容易跑出空指针异常，应使用`常量`或`确定有值`的对象来调用equals。

2.推荐使用java.util.Objects#queals

```java
Object.equals(null,"abc");
//源码
public static boolean equals(obj a,Obj b){
    return (a==b)||(a!=null&&a.equals(b));
}
//通过源码可以看出来，可以避免空指针异常，如果a==null的话，此时，后面的代码就不会执行，避免空指针异常
```

注意：

每种原始类型都有默认值，如int是0，boolean默认值是false，null是所有`引用类型`的默认值，不严格的说，是所有Object类型的默认值

可以使用==或！=来操作比较null值

不能使用一个值为null的引用类型变量来调用非静态方法，否则会抛出异常

### 1.2.整型包装类值的比较

整型包装类值的比较必须使用equals方法

### 1.3.BigDecimal

浮点数之间的等值判断，基本数据类型不能用==来比较，包装数据类型不能用equals来比较（精度丢失）

```java
BigDecimal a= new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
BigDecimal c = new BigDecimal("0.8");

BigDecimal x = a.subtract(b);
BigDecimal y = b.subtract(c);

sout(x);//0.1
sout(y);//0.1
```

#### 1.3.1 BigDecimal 的大小比较

```java
a.comparaTo(b)//返回-1 表示 a 小于 b，0 表示等于，1表示大于
```

#### 1.3.2 保留几位小数

>通过setScale方法设置保留几位小数以及保留规则。保留规则有很多种，无需记

#### 1.3.3  使用注意事项

> 为防止精度丢失，推荐使用参数为String的构造方法，来创建对象

#### 1.3.4 总结

BigDecimal 主要用来操作（大）浮点数，BigInteger 主要用来操作大整数

前者实现利用到了后者，所不同的是前者加入了小数位的概念

### 1.4 基本数据类型和包装数据类型的使用标准

1.所有POJO类属性都必须使用包装数据类型

2.PRC方法返回值和参数必须使用包装数据类型

3.推荐所有的局部变量使用基本数据类型

## 2.集合

### 2.1 Arrays.aslist()使用指南

### 2.1.1 简介

Arrays.asList()可以使用它，将一个数组转换为一个List集合 





