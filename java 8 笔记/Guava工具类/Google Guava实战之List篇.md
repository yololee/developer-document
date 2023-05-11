# Google Guava 实战之List篇

## 一、导入依赖

```xml
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>27.0.1-jre</version>
        </dependency>
```

## 二、List

### Lists.cartesianProduct：返回俩个集合的n元笛卡尔积

```java
  @Test
    public void test1(){
        List<User> users = new ArrayList<>();
        users.add(new User("a", 20));
        users.add(new User("b", 21));
        users.add(new User("c", 22));

        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        /**
         * 1、cartesianProduct
         * Lists.cartesianProduct:通过从每个给定列表中依次选择一个元素，返回可以形成的所有可能列表；列表的“ n元笛卡尔积 ”。
         */
        List<List<Object>> cartesianProduct = Lists.cartesianProduct(users, list);
        System.out.println("Lists.cartesianProduct:===============cartesianProduct:" + cartesianProduct);
    }

    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public static class User{
        private String userName;
        private Integer age;
    }
```

> 输出结果

```java
Lists.cartesianProduct:===============cartesianProduct:[[ListTest.User(userName=a, age=20), 1], [ListTest.User(userName=a, age=20), 2], [ListTest.User(userName=b, age=21), 1], [ListTest.User(userName=b, age=21), 2], [ListTest.User(userName=c, age=22), 1], [ListTest.User(userName=c, age=22), 2]]
```

###  Lists.reverse：返回指定列表的反向视图

```java
 	@Test
    public void test1(){
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        /**
         * 	2、reverse(List<T> list)
         * 返回指定列表的反向视图。
         */
        List<Integer> reverse = Lists.reverse(list);
        System.out.println("Lists.reverse=============revers:" + reverse);
    }
```

> 输出结果

```java
Lists.reverse=============revers:[2, 1]
```

### Lists.partition：把List按指定大小分割

```java
 	@Test
    public void test1(){
        List<User> users = new ArrayList<>();
        users.add(new User("a", 20));
        users.add(new User("b", 21));
        users.add(new User("c", 22));
        /**
         * 3、partition(List, int)
         * 把List按指定大小分割
         */
        List<List<User>> partition = Lists.partition(users, 2);
        System.out.println("Lists.partition:===========partition:" + partition);
    }
```

> 输出结果

```java
Lists.partition:===========partition:[[ListTest.User(userName=a, age=20), ListTest.User(userName=b, age=21)], [ListTest.User(userName=c, age=22)]]
```

### Lists.charactersOf(String string)：返回指定字符串的视图作为不可变Character值列表

```java
 	@Test
    public void test1(){
        /**
         * 5、Lists.charactersOf(String string)
         * 返回指定字符串的视图作为不可变Character值列表。
         */
        ImmutableList<Character> characters = Lists.charactersOf("张三在学guava");
        System.out.println("Lists.charactersOf(String string)=================="+characters);
    }
```

> 输出结果

```java
Lists.charactersOf(String string)==================[张, 三, 在, 学, g, u, a, v, a]
```

### Lists.charactersOf(CharSequence sequence)：返回指定字符串的视图作为不可变Character值列表

```java
 	@Test
    public void test1(){

        /**
         * 6、Lists.charactersOf(CharSequence sequence)
         *返回指定CharSequence为的List<Character>视图sequence，以Unicode代码单元序列的形式查看 。
         */
        List<Character> characters1 = Lists.charactersOf(new StringBuffer("张三在学guava"));
        System.out.println("Lists.charactersOf(CharSequence sequence)=================="+characters1);
    }
```

> 输出结果

```java
Lists.charactersOf(CharSequence sequence)==================[张, 三, 在, 学, g, u, a, v, a]
```