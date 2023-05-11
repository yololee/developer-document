# java中各种类型用Stream流求和

## 一、BigDecimal类型

### 1. 对一个实体类是某一个字段求和

```java
 List<User> userList = new ArrayList<>(Arrays.asList(
                new User(1,new BigDecimal("10")),
                new User(2,new BigDecimal("100"))));

        BigDecimal sum = userList.stream().filter(Objects::nonNull).map(User::getMoney).reduce(BigDecimal.ZERO, BigDecimal::add);
```

### 2. 分组求和

数据准备

```java
YearNum yearNum1 = new YearNum(2021, BigDecimal.valueOf(10));
        YearNum yearNum11 = new YearNum(2021, BigDecimal.valueOf(10));
        YearNum yearNum2 = new YearNum(2022, BigDecimal.valueOf(20));
        YearNum yearNum22 = new YearNum(2022, BigDecimal.valueOf(20));
        List<YearNum> years = ListUtil.of(yearNum1, yearNum11, yearNum2, yearNum22);
```

> 方式一

```java
Map<Integer, BigDecimal> numByYear = years.stream()
                .collect(Collectors.groupingBy(YearNum::getYear,
                        Collectors.reducing(BigDecimal.ZERO, YearNum::getNum, BigDecimal::add)));
```

> 方式二

```java
Map<Integer, BigDecimal> numByYear2 = years.stream().filter(Objects::nonNull).collect(
                Collectors.groupingBy(YearNum::getYear, Collectors.mapping(YearNum::getNum, Collectors.reducing(BigDecimal.ZERO, BigDecimal::add))));
```

## 二、int、double、long类型

### 1. Collectors.summarizingInt()实现

```java
        List<User> userList = new ArrayList<>(Arrays.asList(
                new User(1L,new BigDecimal("10")),
                new User(2L,new BigDecimal("100"))));
        
        DoubleSummaryStatistics collect = userList.stream().collect(Collectors.summarizingDouble(User::getId));
        double sum = collect.getSum();
```

### 2. stream().reduce()实现

> 对象的一个字段求和

```java
        List<User> userList = new ArrayList<>(Arrays.asList(
                new User(1L,new BigDecimal("10")),
                new User(2L,new BigDecimal("100"))));
        Long reduce = userList.stream().filter(Objects::nonNull).map(User::getId).reduce(0L, Long::sum);
```

> Long类型的集合

```java
 		List<Long> list = ListUtil.of(1L,2L);
        Long reduce = list.stream().reduce(0L, Long::sum);
```

### 3. stream().mapToLong()实现

> Long类型的集合

```java
        List<Long> list = ListUtil.of(1L,2L);
        long sum = list.stream().mapToLong(s -> s).sum();
```

> 对象的一个字段求和

```java
        List<User> userList = new ArrayList<>(Arrays.asList(
                new User(1L,new BigDecimal("10")),
                new User(2L,new BigDecimal("100"))));
        long sum = userList.stream().mapToLong(User::getId).sum();
```

