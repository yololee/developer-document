# StringJoiner 详解 (拼接字符串添加分隔符、前缀和后缀)

### StringJoiner(CharSequence d)

此构造函数使用分隔符分隔添加的元素

```java
    @Test
    public void test1(){
        StringJoiner sj = new StringJoiner(",");
        sj.add("aa").add("bb").add("cc");
        System.out.println(sj);
    }
```

输出结果：

```java
aa,bb,cc
```

### StringJoiner(CharSequence d, CharSequence p, CharSequence s)

这个构造函数也需要前缀和后缀来添加。前缀和后缀不取决于添加元素的数量

```java
    @Test
    public void test2(){
        StringJoiner sj = new StringJoiner(",","(",")");
        sj.add("aa").add("bb").add("cc");
        System.out.println(sj);
    }
```

输出结果：

```java
(aa,bb,cc)
```

### StringJoiner.merge(StringJoiner other)

我们可以合并两个`StringJoiner`。将会有一个主要的`StringJoiner`，另一个`StringJoiner`将被添加到其中。

另一个`StringJoiner`在被添加到主`StringJoiner`时不会带来其前缀和后缀。

```java
    @Test
    public void test3(){
        StringJoiner sj1 = new StringJoiner(",","(",")");
        sj1.add("aa").add("bb").add("cc");
        StringJoiner sj2= new StringJoiner(",");
        sj2.add("11").add("22");

        StringJoiner merge = sj2.merge(sj1);
        System.out.println(merge);
    }
```

输出结果：

```java
11,22,aa,bb,cc
```

### StringJoiner.length()

`StringJoiner.length()`像普通的字符串长度方法一样获得长度

```java
    @Test
    public void test4(){
        StringJoiner sjObj = new StringJoiner(",", "{", "}");
        //Add Element
        sjObj.add("AA").add("BB").add("CC").add("DD").add("EE");
        String output = sjObj.toString();
        System.out.println(output);
        //Create another StringJoiner
        StringJoiner otherSj = new StringJoiner(":", "(", ")");
        otherSj.add("10").add("20").add("30");
        System.out.println(otherSj);
        //Use StringJoiner.merge(StringJoiner o)
        StringJoiner finalSj = sjObj.merge(otherSj);
        System.out.println(finalSj);
        //get length using StringJoiner.length()
        System.out.println("Length of Final String:"+finalSj.length());
    }
```

输出结果：

```java
{AA,BB,CC,DD,EE}
(10:20:30)
{AA,BB,CC,DD,EE,10:20:30}
Length of Final String:25
```

