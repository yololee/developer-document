# Google Guava 实战之Map篇

## 一、导入依赖

```xml
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>27.0.1-jre</version>
        </dependency>
```

## 二、Map

### Maps.difference(Map, Map)用来比较两个Map以获取所有不同点

```java
@Test
    public void test1(){
        Map<String, Object> map1 = new HashMap<>();
        map1.put("张三", 123);
        map1.put("李四", 457);
        map1.put("王五", 235);
        map1.put("马六", 752);
        map1.put("王八", 752);

        Map<String, Object> map2 = new HashMap<>();
        map2.put("张三", 345);
        map2.put("马六", 752);
        map2.put("田七", 125);

        /**
         * 1、difference
         * Maps.difference(Map, Map)用来比较两个Map以获取所有不同点。该方法返回MapDifference对象
         */
        MapDifference<String, Object> difference = Maps.difference(map1, map2);
        //是否有差异，返回boolean
        boolean areEqual = difference.areEqual();
        System.out.println("比较两个Map是否有差异:" + areEqual);
        //两个map的交集
        Map<String, Object> entriesInCommon = difference.entriesInCommon();
        System.out.println("两个map都有的部分（交集）===：" + entriesInCommon);
        //键相同但是值不同值映射项。返回的Map的值类型为MapDifference.ValueDifference，以表示左右两个不同的值
        Map<String, MapDifference.ValueDifference<Object>> entriesDiffering = difference.entriesDiffering();
        System.out.println("键相同但是值不同值映射项===：" + entriesDiffering);
        //键只存在于左边Map的映射项
        Map<String, Object> onlyOnLeft = difference.entriesOnlyOnLeft();
        System.out.println("键只存在于左边Map的映射项:" + onlyOnLeft);
        //键只存在于右边Map的映射项
        Map<String, Object> entriesOnlyOnRight = difference.entriesOnlyOnRight();
        System.out.println("键只存在于右边Map的映射项:" + entriesOnlyOnRight);
    }
```

> 输出结果

```java
比较两个Map是否有差异:false
两个map都有的部分（交集）===：{马六=752}
键相同但是值不同值映射项===：{张三=(123, 345)}
键只存在于左边Map的映射项:{李四=457, 王五=235, 王八=752}
键只存在于右边Map的映射项:{田七=125}
```

### Maps.filterEntries():过滤map

```java
@Test
    public void test2(){
        Map<String, Object> map1 = new HashMap<>();
        map1.put("张三", 123);
        map1.put("李四", 457);
        map1.put("王五", 235);
        map1.put("马六", 752);
        map1.put("王八", 752);

        /**
         * 2.filterEntries
         * Maps.filterEntries():过滤map
         */
        Map<String, Object> filterEntries = Maps.filterEntries(map1, input -> {
            if (input.getValue().equals(123) && Objects.equals(input.getKey(), "张三")) {
                return true;
            }
            return false;
        });
        System.out.println("过滤map=====filterEntries:" + filterEntries);
    }
```

> 输出结果

```java
过滤map=====filterEntries:{张三=123}
```

### Maps.filterKeys():过滤key

```java
 @Test
    public void test3(){
        Map<String, Object> map1 = new HashMap<>();
        map1.put("张三", 123);
        map1.put("李四", 457);
        map1.put("王五", 235);
        map1.put("马六", 752);
        map1.put("王八", 752);

        /**
         * 3.filterKeys
         * Maps.filterKeys():过滤key
         */
        Map<String, Object> filterKeysMap = Maps.filterKeys(map1, input -> input.equals("李四"));
        System.out.println("过滤key=====filterKeys:" + filterKeysMap);
    }
```

> 输出结果

```java
过滤key=====filterKeys:{李四=457}
```

### Maps.filterValues():过滤value

```java
 @Test
    public void test4(){
        Map<String, Object> map1 = new HashMap<>();
        map1.put("张三", 123);
        map1.put("李四", 457);
        map1.put("王五", 235);
        map1.put("马六", 752);
        map1.put("王八", 752);

        /**
         *4.filterValues
         * Maps.filterValues():过滤value
         */
        Map<String, Object> filterValuesMap = Maps.filterValues(map1, input -> Objects.equals(752, input));
        System.out.println("过滤value=====filterValues:" + filterValuesMap);
    }
```

> 输出结果

```java
过滤value=====filterValues:{马六=752, 王八=752}
```

### Maps.uniqueIndex():根据属性值找对象

```java
 @Test
    public void test4(){
        /**
         * 5.uniqueIndex
         * Maps.uniqueIndex():根据属性值找对象
         * 有一组对象，它们在某个属性上分别有独一无二的值，而我们希望能够按照这个属性值查找对象
         * 注：Maps.uniqueIndex():必须属性值唯一
         *否则报错：Exception in thread "main" java.lang.IllegalArgumentException: Multiple entries with same key:xxxxx. To index multiple values under a key, use Multimaps.index.
         */
        List<User> users = new ArrayList<>();
        users.add(new User("zhangsan", 24));
        users.add(new User("lisi", 27));
        users.add(new User("wangwu", 28));
//        users.add(new User("wangwu",30));
        ImmutableMap<Object, User> uniqueIndex = Maps.uniqueIndex(users, new Function<User, Object>() {
            @Nullable
            @Override
            public Object apply(@Nullable User user) {
                return user.getUserName();
            }
        });
        System.out.println("Maps.uniqueIndex()===========uniqueIndex:" + uniqueIndex);
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
Maps.uniqueIndex()===========uniqueIndex:{zhangsan=MapTest.User(userName=zhangsan, age=24), lisi=MapTest.User(userName=lisi, age=27), wangwu=MapTest.User(userName=wangwu, age=28)}
```

### Multimaps.index(): 根据属性值找对象

```java
	@Test
    public void test4(){

        List<User> users = new ArrayList<>();
        users.add(new User("zhangsan", 24));
        users.add(new User("lisi", 27));
        users.add(new User("wangwu", 28));
        users.add(new User("wangwu", 30));
        users.add(new User("lisi", 23));

        /**
         * 6.index
         * Multimaps.index(): 根据属性值找对象
         * 如果您的索引可能将多个值与每个键相关联
         */
        ImmutableListMultimap<Object, User> multimap = Multimaps.index(users, new Function<User, Object>() {
            @Nullable
            @Override
            public Object apply(@Nullable User user) {
                return user.getUserName();
            }
        });
        System.out.println("Multimaps.index()=========multimap:" + multimap);
    }
```

> 输出结果

```java
Multimaps.index()=========multimap:{zhangsan=[MapTest.User(userName=zhangsan, age=24)], lisi=[MapTest.User(userName=lisi, age=27), MapTest.User(userName=lisi, age=23)], wangwu=[MapTest.User(userName=wangwu, age=28), MapTest.User(userName=wangwu, age=30)]}

```

### Maps.immutableEntry():返回不可变的映射条目与指定的键和值

```java
	@Test
    public void test4(){
        /**
         * 7.immutableEntry
         * Maps.immutableEntry():返回不可变的映射条目与指定的键和值
         */
        Map.Entry<String, Integer> immutableEntry = Maps.immutableEntry("李四", 457);
        System.out.println("Maps.immutableEntry:===============immutableEntry:" + immutableEntry);


    }
```

> 输出结果

```java
Maps.immutableEntry:===============immutableEntry:李四=457
```

### Maps.subMap():返回的部分视图， map的键被包含range限制

```java
	@Test
    public void test4(){
        /**
         * 8.subMap
         * Maps.subMap():返回的部分视图， map的键被包含range限制
         */
        TreeMap<Integer, String> TreeMap = Maps.newTreeMap();
        TreeMap.put(1, "aaa");
        TreeMap.put(4, "bbb");
        TreeMap.put(2, "ccc");
        TreeMap.put(6, "ddd");
        //返回包含严格大于所有值的范围lower ，比严格小于uppe
        Range<Integer> range = Range.open(0, 3);
        NavigableMap<Integer, String> navigableMap = Maps.subMap(TreeMap, range);
        System.out.println("Maps.subMap()=================navigableMap:" + navigableMap);

    }
```

> 输出结果

```java
Maps.subMap()=================navigableMap:{1=aaa, 2=ccc}
```

### Maps.transformValues:返回一个map，其中每个值都通过函数进行转换

```java
    @Test
	public void test4(){

        Map<String, Object> map1 = new HashMap<>();
        map1.put("张三", 123);
        map1.put("李四", 457);
        map1.put("王五", 235);
        map1.put("马六", 752);
        map1.put("王八", 752);

        /**
         * 9.transformValues
         * Maps.transformValues:返回一个map，其中每个值都通过函数进行转换。
         */
        Map<String, Object> transformValues = Maps.transformValues(map1, new Function<Object, Object>() {
            @Nullable
            @Override
            public Object apply(@Nullable Object input) {
                Integer value = Integer.valueOf(input.toString());
                return value / 2;
            }
        });
        System.out.println("Maps.transformValues===============transformValues:" + transformValues);

    }
```

> 输出结果

```java
Maps.transformValues===============transformValues:{李四=228, 张三=61, 马六=376, 王五=117, 王八=376}
```