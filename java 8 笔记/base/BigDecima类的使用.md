# BigDecimal类的使用

### 1、简介

Java在java.math包中提供的API类BigDecimal，用来对超过16位有效位的数进行精确的运算。双精度浮点型变量double可以处理16位有效数。在实际应用中，需要对更大或者更小的数进行运算和处理。float和double只能用来做科学计算或者是工程计算，在商业计算中要用java.math.BigDecimal。BigDecimal所创建的是对象，我们不能使用传统的+、-、*、/等算术运算符直接对其对象进行数学运算，而必须调用其相对应的方法。方法中的参数也必须是BigDecimal的对象。构造器是类的特殊方法，专门用来创建对象，特别是带有参数的对象。

### 2、构造方法

| 方法               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| BigDecimal(int)    | 创建一个具有参数所指定整数值的对象。                         |
| BigDecimal(double) | 创建一个具有参数所指定双精度值的对象。 <font color= 'red'>不推荐使用</font> |
| BigDecimal(long)   | 创建一个具有参数所指定长整数值的对象。                       |
| BigDecimal(String) | 创建一个具有参数所指定以字符串表示的数值的对象。<font color= 'red'>推荐使用</font> |

### 3、方法描述

| 方法                 | 描述                                         |
| -------------------- | -------------------------------------------- |
| add(BigDecimal)      | BigDecimal对象中的值相加，然后返回这个对象。 |
| subtract(BigDecimal) | BigDecimal对象中的值相减，然后返回这个对象。 |
| multiply(BigDecimal) | BigDecimal对象中的值相乘，然后返回这个对象。 |
| divide(BigDecimal)   | BigDecimal对象中的值相除，然后返回这个对象。 |
| toString()           | 将BigDecimal对象的数值转换成字符串。         |
| doubleValue()        | 将BigDecimal对象中的值以双精度数返回。       |
| floatValue()         | 将BigDecimal对象中的值以单精度数返回。       |
| longValue()          | 将BigDecimal对象中的值以长整数返回。         |
| intValue()           | 将BigDecimal对象中的值以整数返回。           |

### 4、构造方法为什么不推荐使用BigDecimal(double)

> 测试用例

```java
@Test
    public void testBigDecimal(){
        BigDecimal stringBig = new BigDecimal("22");
        BigDecimal doubleBig = new BigDecimal(1.111111111111111111111);
        System.out.println(stringBig);
        System.out.println(doubleBig);
    }
```

> 输出结果

```java
22
1.111111111111111160454356650006957352161407470703125
```

**为什么会出现这种情况呢？**

1. 参数类型为double的构造方法的结果有一定的不可预知性。有人可能认为在Java中写入newBigDecimal(0.1)所创建的BigDecimal正好等于 0.1（非标度值 1，其标度为 1），但是它实际上等于0.1000000000000000055511151231257827021181583404541015625。这是因为0.1无法准确地表示为 double（或者说对于该情况，不能表示为任何有限长度的二进制小数）。这样，传入到构造方法的值不会正好等于 0.1（虽然表面上等于该值）。
2. String 构造方法是完全可预知的：写入 newBigDecimal("0.1") 将创建一个 BigDecimal，它正好等于预期的 0.1。因此，比较而言，**通常建议优先使用String构造方法**

### 5、加减乘除

```java
@Test
    public void testBigDecimal2(){
        //使用double形式初始化
        BigDecimal num1 = new BigDecimal(0.005);
        BigDecimal num2 = new BigDecimal(1000000);
        BigDecimal num3 = new BigDecimal(-1000000);
        //尽量用字符串的形式初始化
        BigDecimal num12 = new BigDecimal("0.005");
        BigDecimal num22 = new BigDecimal("1000000");
        BigDecimal num32 = new BigDecimal("-1000000");

        //加法
        //加法
        BigDecimal result1 = num1.add(num2);
        BigDecimal result12 = num12.add(num22);

        //减法
        BigDecimal result2 = num1.subtract(num2);
        BigDecimal result22 = num12.subtract(num22);

        //乘法
        BigDecimal result3 = num1.multiply(num2);
        BigDecimal result32 = num12.multiply(num22);

        //绝对值
        BigDecimal result4 = num3.abs();
        BigDecimal result42 = num32.abs();

        //除法
        BigDecimal result5 = num2.divide(num1,20,BigDecimal.ROUND_HALF_UP);
        BigDecimal result52 = num22.divide(num12,20,BigDecimal.ROUND_HALF_UP);
        
        System.out.println("加法用double结果："+result1);
        System.out.println("加法用String结果："+result12);
        System.out.println("减法用double结果："+result2);
        System.out.println("减法用String结果："+result22);
        System.out.println("乘法用double结果："+result3);
        System.out.println("乘法用String结果："+result32);
        System.out.println("绝对值用double结果："+result4);
        System.out.println("绝对值用String结果："+result42);
        System.out.println("除法用double结果："+result5);
        System.out.println("除法用String结果："+result52);
    }
```

> 输出结果

```java
加法用double结果：1000000.005000000000000000104083408558608425664715468883514404296875
加法用String结果：1000000.005
    
减法用double结果：-999999.994999999999999999895916591441391574335284531116485595703125
减法用String结果：-999999.995
    
乘法用double结果：5000.000000000000104083408558608425664715468883514404296875000000
乘法用String结果：5000.000
    
绝对值用double结果：1000000
绝对值用String结果：1000000
    
除法用double结果：199999999.99999999583666365766
除法用String结果：200000000.00000000000000000000
```

### 6、使用除法的注意事项

使用除法函数divide的时候要设置各种参数，<font color='red'>要精确到小位数和舍入模式</font>，不然会报错

divide的配置参数：

```java
public BigDecimal divide(BigDecimal divisor,int scale,int roundingMode)
    -BigDecimal diviso 除数
    -int scale 精确小数位
    -int roundingMode 舍入模式
```

> 八种舍入模式

**1、ROUND_UP**

舍入远离零的舍入模式。

在丢弃非零部分之前始终增加数字(始终对非零舍弃部分前面的数字加1)。

注意，此舍入模式始终不会减少计算值的大小。

**2、ROUND_DOWN**

接近零的舍入模式。

在丢弃某部分之前始终不增加数字(从不对舍弃部分前面的数字加1，即截短)。

注意，此舍入模式始终不会增加计算值的大小。

**3、ROUND_CEILING**

接近正无穷大的舍入模式。

如果 BigDecimal 为正，则舍入行为与 ROUND_UP 相同;

如果为负，则舍入行为与 ROUND_DOWN 相同。

注意，此舍入模式始终不会减少计算值。

**4、ROUND_FLOOR**

接近负无穷大的舍入模式。

如果 BigDecimal 为正，则舍入行为与 ROUND_DOWN 相同;

如果为负，则舍入行为与 ROUND_UP 相同。

注意，此舍入模式始终不会增加计算值。

**5、ROUND_HALF_UP**

向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则为向上舍入的舍入模式。

如果舍弃部分 >= 0.5，则舍入行为与 ROUND_UP 相同;否则舍入行为与 ROUND_DOWN 相同。

注意，这是我们大多数人在小学时就学过的舍入模式(四舍五入)。

**6、ROUND_HALF_DOWN**

向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则为上舍入的舍入模式。

如果舍弃部分 > 0.5，则舍入行为与 ROUND_UP 相同;否则舍入行为与 ROUND_DOWN 相同(五舍六入)。

**7、ROUND_HALF_EVEN**

向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则向相邻的偶数舍入。

如果舍弃部分左边的数字为奇数，则舍入行为与 ROUND_HALF_UP 相同;

如果为偶数，则舍入行为与 ROUND_HALF_DOWN 相同。

注意，在重复进行一系列计算时，此舍入模式可以将累加错误减到最小。

此舍入模式也称为“银行家舍入法”，主要在美国使用。四舍六入，五分两种情况。

如果前一位为奇数，则入位，否则舍去。

以下例子为保留小数点1位，那么这种舍入方式下的结果。

1.15>1.2 1.25>1.2

**8、ROUND_UNNECESSARY**

断言请求的操作具有精确的结果，因此不需要舍入。

如果对获得精确结果的操作指定此舍入模式，则抛出ArithmeticException。

> 测试用例

```java
 @Test
    public void testBigDecimal03(){
        BigDecimal num1 = new BigDecimal("1");
        BigDecimal num2 = new BigDecimal("3");
        BigDecimal divide1 = num1.divide(num2, 3, BigDecimal.ROUND_UP);
        BigDecimal divide2 = num1.divide(num2, 3, BigDecimal.ROUND_DOWN);
        BigDecimal divide3 = num1.divide(num2, 3, BigDecimal.ROUND_CEILING);
        BigDecimal divide4 = num1.divide(num2, 3, BigDecimal.ROUND_FLOOR);
        BigDecimal divide5 = num1.divide(num2, 3, BigDecimal.ROUND_HALF_UP);
        BigDecimal divide6 = num1.divide(num2, 3, BigDecimal.ROUND_HALF_DOWN);
        BigDecimal divide7 = num1.divide(num2, 3, BigDecimal.ROUND_HALF_EVEN);
//        BigDecimal divide8 = num1.divide(num2, 3, BigDecimal.ROUND_UNNECESSARY);

        System.out.println(divide1);
        System.out.println(divide2);
        System.out.println(divide3);
        System.out.println(divide4);
        System.out.println(divide5);
        System.out.println(divide6);
        System.out.println(divide7);
//        System.out.println(divide8);
    }
```

> 输出结果

```java
0.334
0.333
0.334
0.333
0.333
0.333
0.333
```