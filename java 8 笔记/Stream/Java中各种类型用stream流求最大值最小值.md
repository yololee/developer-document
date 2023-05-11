# Java中各种类型用Stream流求最大值最小值

## 一、BigDecimal 求最大值和最小值

### 1. stream().reduce()实现

```java
        List<BigDecimal> list = new ArrayList<>(Arrays.asList(new BigDecimal("1"), new BigDecimal("2")));
        BigDecimal max = list.stream().reduce(list.get(0), BigDecimal::max);
        BigDecimal min = list.stream().reduce(list.get(0), BigDecimal::min);
```

### 2. stream().max()或stream().min()实现

```java
        List<BigDecimal> list = new ArrayList<>(Arrays.asList(new BigDecimal("1"), new BigDecimal("2")));
        BigDecimal max = list.stream().max(Comparator.comparing(x -> x)).orElse(null);
        BigDecimal min = list.stream().min(Comparator.comparing(x -> x)).orElse(null);
```

## 二、Integer 求最大值和最小值

### 1. stream().reduce()实现

```java
        List<Integer> list = new ArrayList<>(Arrays.asList(1, 2));
        Integer max = list.stream().reduce(list.get(0), Integer::max);
        Integer min = list.stream().reduce(list.get(0), Integer::min);
```

### 2. Collectors.summarizingInt()实现

```java
        List<Integer> list = new ArrayList<>(Arrays.asList(1, 2));
        IntSummaryStatistics intSummaryStatistics = list.stream().collect(Collectors.summarizingInt(x -> x));
        Integer max = intSummaryStatistics.getMax();
        Integer min = intSummaryStatistics.getMin();
```

### 3. stream().max()或stream().min()实现

```java
        List<Integer> list = new ArrayList<>(Arrays.asList(1, 2));
        Integer max = list.stream().max(Comparator.comparing(x -> x)).orElse(null);
        Integer min = list.stream().min(Comparator.comparing(x -> x)).orElse(null);
```

## 三、Long 求最大值和最小值

### 1. stream().reduce()实现 

```java
        List<Long> list = new ArrayList<>(Arrays.asList(1L, 2L));
        Long max = list.stream().reduce(list.get(0), Long::max);
        Long min = list.stream().reduce(list.get(0), Long::min);
```

### 2. Collectors.summarizingLong()实现

```java
        List<Long> list = new ArrayList<>(Arrays.asList(1L, 2L));
        LongSummaryStatistics summaryStatistics = list.stream().collect(Collectors.summarizingLong(x -> x));
        Long max = summaryStatistics.getMax();
        Long min = summaryStatistics.getMin();
```

### 3. stream().max()或stream().min()实现

```java
        List<Long> list = new ArrayList<>(Arrays.asList(1L, 2L));
        Long max = list.stream().max(Comparator.comparing(x -> x)).orElse(null);
        Long min = list.stream().min(Comparator.comparing(x -> x)).orElse(null);
```

## 四、Double 求最大值和最小值

### 1. stream().reduce()实现 

```java
        List<Double> list = new ArrayList<>(Arrays.asList(1d, 2d));
        Double max = list.stream().reduce(list.get(0), Double::max);
        Double min = list.stream().reduce(list.get(0), Double::min);
```

### 2. Collectors.summarizingLong()实现

```java
        List<Double> list = new ArrayList<>(Arrays.asList(1d, 2d));
        DoubleSummaryStatistics summaryStatistics = list.stream().collect(Collectors.summarizingDouble(x -> x));
        Double max = summaryStatistics.getMax();
        Double min = summaryStatistics.getMin();
```

### 3. stream().max()或stream().min()实现

```java
        List<Double> list = new ArrayList<>(Arrays.asList(1d, 2d));
        Double max = list.stream().max(Comparator.comparing(x -> x)).orElse(null);
        Double min = list.stream().min(Comparator.comparing(x -> x)).orElse(null);
```