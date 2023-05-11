# Java中用stream流进行去重、排序、分组

## 一、distinct

### 1. 八大基本数据类型

```java
        List<Integer> collect = ListUtil.of(1, 2, 3, 1, 2).stream().filter(Objects::nonNull).distinct().collect(Collectors.toList());
        System.out.println(collect);

        List<String> collect1 = ListUtil.of("user1", "user1", "user2").stream().filter(Objects::nonNull).distinct().collect(Collectors.toList());
        System.out.println(collect1);
```

**输出结果：**

```java
[1, 2, 3]
[user1, user2]
```

### 2.根据 `List<Object>` 中 Object 某个属性去重

> 根据一个属性

```java
        List<User> userList = new ArrayList<>(Arrays.asList(
                new User(1L,new BigDecimal("10"),18),
                new User(1L,new BigDecimal("20"),19),
                new User(2L,new BigDecimal("100"),18)));
        System.out.println("去重前"+userList);

        userList = userList.stream().collect(Collectors.collectingAndThen(Collectors.toCollection(() -> new TreeSet<>(Comparator.comparing(o -> o.getId()))), ArrayList::new));

        System.out.println("去重后"+userList);
```

**输出结果：**

```java
去重前[User{id=1, money=10}, User{id=1, money=20}, User{id=2, money=100}]
去重后[User{id=1, money=10}, User{id=2, money=100}]
```

> 根据多个属性

```java
        List<User> userList = new ArrayList<>(Arrays.asList(
                new User(1L,new BigDecimal("10"),18),
                new User(1L,new BigDecimal("20"),19),
                new User(2L,new BigDecimal("100"),18)));
        System.out.println("去重前"+userList);

        userList = userList.stream().collect(Collectors.collectingAndThen(Collectors.toCollection(() -> new TreeSet<>(Comparator.comparing(o -> o.getId() + ";" + o.getAge()))), ArrayList::new));

        System.out.println("去重后"+userList);
```

**输出结果：**

```java
去重前[User{id=1, money=10, age=18}, User{id=2, money=20, age=19}, User{id=3, money=100, age=17}]
去重后[User{id=1, money=10, age=18}, User{id=2, money=20, age=19}, User{id=3, money=100, age=17}]
```

## 二、sorted

### 1. 升序

```java
List<User> userList = new ArrayList<>(Arrays.asList(
                new User(1L,new BigDecimal("10"),18),
                new User(2L,new BigDecimal("20"),19),
                new User(3L,new BigDecimal("100"),17)));
        System.out.println("排序前"+userList);

        userList = userList.stream().sorted(Comparator.comparing(User::getAge)).collect(Collectors.toList());

        System.out.println("排序后"+userList);
```

**输出结果：**

```java
排序前[User{id=1, money=10, age=18}, User{id=2, money=20, age=19}, User{id=3, money=100, age=17}]
排序后[User{id=3, money=100, age=17}, User{id=1, money=10, age=18}, User{id=2, money=20, age=19}]
```

### 2. 倒序

```java
        List<User> userList = new ArrayList<>(Arrays.asList(
                new User(1L,new BigDecimal("10"),18),
                new User(2L,new BigDecimal("20"),19),
                new User(3L,new BigDecimal("100"),17)));
        System.out.println("排序前"+userList);

        userList = userList.stream().sorted(Comparator.comparing(User::getAge).reversed()).collect(Collectors.toList());

        System.out.println("排序后"+userList);
```

**输出结果：**

```java
排序前[User{id=1, money=10, age=18}, User{id=2, money=20, age=19}, User{id=3, money=100, age=17}]
排序后[User{id=2, money=20, age=19}, User{id=1, money=10, age=18}, User{id=3, money=100, age=17}]
```

### 3. 多条件排序

```java
        List<User> userList = new ArrayList<>(Arrays.asList(
                new User(1L,new BigDecimal("10"),18),
                new User(2L,new BigDecimal("20"),19),
                new User(3L,new BigDecimal("100"),18)));
        System.out.println("排序前"+userList);

        userList = userList.stream().sorted(Comparator.comparing(User::getAge).thenComparing(User::getId)).collect(Collectors.toList());

        System.out.println("排序后"+userList);
```

**输出结果：**

```java
排序前[User{id=1, money=10, age=18}, User{id=2, money=20, age=19}, User{id=3, money=100, age=18}]
排序后[User{id=1, money=10, age=18}, User{id=3, money=100, age=18}, User{id=2, money=20, age=19}]
```
### 4. map排序
```java
//根据时间进行升序排序
Map<String, String> map = new HashMap<>();
Map<String, String> result = new LinkedHashMap<>();
map.entrySet().stream().sorted(Map.Entry.comparingByKey()).forEachOrdered(x -> result.put(DateUtil.timeStamp2Date(x.getKey(), null), x.getValue()));   


//根据时间进行降序排序
Map<String, String> map = new HashMap<>();
Map<String, String> result = new LinkedHashMap<>();
map.entrySet().stream().sorted(Collections.reverseOrder(Map.Entry.comparingByKey())).forEachOrdered(x -> result.put(DateUtil.timeStamp2Date(x.getKey(), null), x.getValue()));  
```

## 三、groupingBy

### 1. 根据一个字段进行分组

```java
        List<User> userList = new ArrayList<>(Arrays.asList(
                new User(1L,new BigDecimal("10"),18),
                new User(2L,new BigDecimal("20"),19),
                new User(3L,new BigDecimal("100"),18)));

        Map<Integer, List<User>> collect = userList.stream().filter(Objects::nonNull).collect(Collectors.groupingBy(User::getAge));
        System.out.println(collect);
```

**输出结果：**

```java
{18=[User{id=1, money=10, age=18}, User{id=3, money=100, age=18}], 19=[User{id=2, money=20, age=19}]}
```

### 2. 根据俩个字段进行分组

```java
        List<User> userList = new ArrayList<>(Arrays.asList(
                new User(1L,new BigDecimal("10"),18),
                new User(2L,new BigDecimal("20"),19),
                new User(3L,new BigDecimal("100"),18)));

        Map<Integer, Map<BigDecimal, List<User>>> collect = userList.stream().filter(Objects::nonNull)
                .collect(Collectors.groupingBy(User::getAge, Collectors.groupingBy(User::getMoney)));

        System.out.println(JSONUtil.toJsonStr(collect));
```

**输出结果：**

![image-20230511092848384](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511092848384.png)


### 3. 复杂分组

```java
        List<User> userList = new ArrayList<>(Arrays.asList(
                new User(1L,new BigDecimal("10"),18),
                new User(2L,new BigDecimal("20"),19),
                new User(3L,new BigDecimal("100"),18)));

        Map<Integer, Map<BigDecimal, List<Long>>> collect = userList.stream().filter(Objects::nonNull)
                .collect(Collectors.groupingBy(User::getAge, Collectors.groupingBy(User::getMoney, Collectors.mapping(User::getId, Collectors.toList()))));

        System.out.println(JSONUtil.toJsonStr(collect));
```

**输出结果：**

![image-20230511092900310](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511092900310.png)