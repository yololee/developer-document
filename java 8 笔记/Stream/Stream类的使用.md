# Stream

![image-20230511092019532](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511092019532.png)


## 1、Stream的创建

1、通过集合创建流

```java
List<String> list = Arrays.asList("a", "b", "c");
// 创建一个顺序流
Stream<String> stream = list.stream();
// 创建一个并行流
Stream<String> parallelStream = list.parallelStream();
```

2、通过数组创建流

```java
int[] array={1,3,5,6,8};
IntStream stream = Arrays.stream(array);
```

3、Stream的静态方法 `of()、iterate()、generate()`

```java
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5, 6);

Stream<Integer> stream2 = Stream.iterate(0, (x) -> x + 3).limit(4);
stream2.forEach(System.out::println);
//输出结果：0 3 6 9

Stream<Double> stream3 = Stream.generate(Math::random).limit(3);
stream3.forEach(System.out::println);
//输出结果：0.6796156909271994   0.1914314208854283    0.8116932592396652
```

## 2、Stream的使用

> 案例使用的员工类

```java
@Data
public class Person {
	private String name;  // 姓名
	private int salary; // 薪资
	private int age; // 年龄
	private String sex; //性别
	private String area;  // 地区

	public Person(String name, int salary, String sex, String area) {
		this.name = name;
		this.salary = salary;
		this.sex = sex;
		this.area = area;
	}
}

    private List<Person> getData(){
        List<Person> personList = new ArrayList<Person>();
        personList.add(new Person("Tom", 8900, "male", "New York"));
        personList.add(new Person("Jack", 7000, "male", "Washington"));
        personList.add(new Person("Lily", 7800, "female", "Washington"));
        personList.add(new Person("Anni", 8200, "female", "New York"));
        personList.add(new Person("Owen", 9500, "male", "New York"));
        personList.add(new Person("Alisa", 7900, "female", "New York"));
        return personList;
    }
```

### 2.1、遍历/匹配（foreach/find/match）

```java
        List<Integer> list = Arrays.asList(7, 6, 9, 3, 8, 2, 1);

        // 遍历输出符合条件的元素   输出结果：7,8,9
        list.stream().filter(x -> x > 6).forEach(System.out::println);
        // 匹配第一个
        Optional<Integer> findFirst = list.stream().filter(x -> x > 6).findFirst();
        // 匹配任意（适用于并行流）
				// 只要在任何片段发现了第一个匹配元素就会结束整个运算
        Optional<Integer> findAny = list.parallelStream().filter(x -> x > 6).findAny();
        // 是否包含符合特定条件的元素
        boolean anyMatch = list.stream().anyMatch(x -> x > 6);
        System.out.println("匹配第一个值：" + findFirst.get());//输出结果：7
        System.out.println("匹配任意一个值：" + findAny.get());//输出结果：7
        System.out.println("是否存在大于6的值：" + anyMatch);//输出结果：true
```

### 2.2 、 筛选（filter）

```java
        // 筛选出集合中大于三小于8的元素
        Integer[] arr = new Integer[]{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
        Arrays.stream(arr).filter(x -> x > 3 && x < 8).forEach(System.out::println);
        //输出结果 4,5,6,7

        //筛选员工中工资高于8000的人
        List<Person> personList = new ArrayList<>();
        personList.add(new Person("Tom", 8900, 23, "male", "New York"));
        personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
        personList.add(new Person("Lily", 7800, 21, "female", "Washington"));
        personList.add(new Person("Anni", 8200, 24, "female", "New York"));
        personList.add(new Person("Owen", 9500, 25, "male", "New York"));
        personList.add(new Person("Alisa", 7900, 26, "female", "New York"));

        List<String> fiterList = personList.stream().filter(x -> x.getSalary() > 8000).map(Person::getName).collect(Collectors.toList());
        System.out.print("薪资高于8000美元的员工：" + fiterList);
```

### 2.3、聚合（max/min/count)

> **获取`String`集合中最长的元素**

```java
        List<String> list = Arrays.asList("adnm", "admmt", "pot", "xbangd", "weoujgsd");

        Optional<String> max = list.stream().max(Comparator.comparing(String::length));
        System.out.println("最长的字符串：" + max.get());
				//输出结果：最长的字符串：weoujgsd
```

> **获取`Integer`集合中的最大值**

```java
        List<Integer> list = Arrays.asList(7, 6, 9, 4, 11, 6);

        // 自然排序
        Optional<Integer> max = list.stream().max(Integer::compareTo);
        // 自定义排序（从大到小排序）
        Optional<Integer> max2 = list.stream().max((o1, o2) -> o2 - o1);
        System.out.println("自然排序的最大值：" + max.get());
        //输出结果：自然排序的最大值：11
        System.out.println("自定义排序的最大值：" + max2.get());
        //输出结果：自定义排序的最大值：4
```

> **获取员工薪资最高的人**

```java
        List<Person> personList = new ArrayList<Person>();
        personList.add(new Person("Tom", 8900, 23, "male", "New York"));
        personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
        personList.add(new Person("Lily", 7800, 21, "female", "Washington"));
        personList.add(new Person("Anni", 8200, 24, "female", "New York"));
        personList.add(new Person("Owen", 9500, 25, "male", "New York"));
        personList.add(new Person("Alisa", 7900, 26, "female", "New York"));

        int salary = personList.stream().max(Comparator.comparingInt(Person::getSalary)).get().getSalary();
        System.out.println("员工薪资最大值：" + salary);
        //输出结果：员工薪资最大值：9500
```

> **计算`Integer`集合中大于6的元素的个数**

```java
        List<Integer> list = Arrays.asList(7, 6, 4, 8, 2, 11, 9);

        long count = list.stream().filter(x -> x > 6).count();
        System.out.println("list中大于6的元素个数：" + count);
        //输出结果：list中大于6的元素个数：4
```

### 2.4 映射(map/flatMap)

映射，可以将一个流的元素按照一定的映射规则映射到另一个流中。分为`map`和`flatMap`：

- `map`：接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素
- `flatMap`：接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流

> **英文字符串数组的元素全部改为大写。整数数组每个元素+3**

```java
        String[] strArr = { "abcd", "bcdd", "defde", "fTr" };
        List<String> strList = Arrays.stream(strArr).map(String::toUpperCase).collect(Collectors.toList());

        List<Integer> intList = Arrays.asList(1, 3, 5, 7, 9, 11);
        List<Integer> intListNew = intList.stream().map(x -> x + 3).collect(Collectors.toList());

        System.out.println("每个元素大写：" + strList);
        //输出结果：每个元素大写：[ABCD, BCDD, DEFDE, FTR]
        System.out.println("每个元素+3：" + intListNew);
        //输出结果：每个元素+3：[4, 6, 8, 10, 12, 14]
```

> **将员工的薪资全部增加10000**

```java
        List<Person> personList = new ArrayList<Person>();
        personList.add(new Person("Tom", 8900, 23, "male", "New York"));
        personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
        personList.add(new Person("Lily", 7800, 21, "female", "Washington"));
        personList.add(new Person("Anni", 8200, 24, "female", "New York"));
        personList.add(new Person("Owen", 9500, 25, "male", "New York"));
        personList.add(new Person("Alisa", 7900, 26, "female", "New York"));

        // 不改变原来员工集合的方式
        List<Person> personListNew = personList.stream().map(person -> {
            Person personNew = new Person(person.getName(), 0, 0, null, null);
            personNew.setSalary(person.getSalary() + 10000);
            return personNew;
        }).collect(Collectors.toList());
        System.out.println("一次改动前：" + personList.get(0).getName() + "-->" + personList.get(0).getSalary());
        System.out.println("一次改动后：" + personListNew.get(0).getName() + "-->" + personListNew.get(0).getSalary());

        // 改变原来员工集合的方式
        List<Person> personListNew2 = personList.stream().map(person -> {
            person.setSalary(person.getSalary() + 10000);
            return person;
        }).collect(Collectors.toList());
        System.out.println("二次改动前：" + personList.get(0).getName() + "-->" + personListNew.get(0).getSalary());
        System.out.println("二次改动后：" + personListNew2.get(0).getName() + "-->" + personListNew.get(0).getSalary());
```

输出结果：

```
一次改动前：Tom–>8900
一次改动后：Tom–>18900
二次改动前：Tom–>18900
二次改动后：Tom–>18900
```

> **将两个字符数组合并成一个新的字符数组**

```java
        List<String> list = Arrays.asList("m,k,l,a", "1,3,5,7");
        List<String> listNew = list.stream().flatMap(s -> {
            // 将每个元素转换成一个stream
            String[] split = s.split(",");
            Stream<String> s2 = Arrays.stream(split);
            return s2;
        }).collect(Collectors.toList());

        System.out.println("处理前的集合：" + list);
        //输出结果：处理前的集合：[m-k-l-a, 1-3-5]
        System.out.println("处理后的集合：" + listNew);
        ////输出结果：处理后的集合：[m, k, l, a, 1, 3, 5]
```

### 2.5、归约(reduce)

归约，也称缩减，顾名思义，是把一个流缩减成一个值，能实现对集合求和、求乘积和求最值操作

> **求`Integer`集合的元素之和、乘积和最大值**

```java
        List<Integer> list = Arrays.asList(1, 3, 2, 8, 11, 4);
        // 求和方式1
        Optional<Integer> sum = list.stream().reduce((x, y) -> x + y);
        // 求和方式2
        Optional<Integer> sum2 = list.stream().reduce(Integer::sum);
        // 求和方式3
        Integer sum3 = list.stream().reduce(0, Integer::sum);

        // 求乘积
        Optional<Integer> product = list.stream().reduce((x, y) -> x * y);

        // 求最大值方式1
        Optional<Integer> max = list.stream().reduce((x, y) -> x > y ? x : y);
        // 求最大值写法2
        Integer max2 = list.stream().reduce(1, Integer::max);

        System.out.println("list求和：" + sum.get() + "," + sum2.get() + "," + sum3);
        //输出结果：list求和：29,29,29
        System.out.println("list求积：" + product.get());
        //输出结果：list求积：2112
        System.out.println("list求最大值：" + max.get() + "," + max2);
        //输出结果：list求最大值：11,11
```

> **求所有员工的工资之和和最高工资**

```java
        List<Person> personList = new ArrayList<Person>();
        personList.add(new Person("Tom", 8900, 23, "male", "New York"));
        personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
        personList.add(new Person("Lily", 7800, 21, "female", "Washington"));
        personList.add(new Person("Anni", 8200, 24, "female", "New York"));
        personList.add(new Person("Owen", 9500, 25, "male", "New York"));
        personList.add(new Person("Alisa", 7900, 26, "female", "New York"));

        // 求工资之和方式1：
        Optional<Integer> sumSalary = personList.stream().map(Person::getSalary).reduce(Integer::sum);
        // 求工资之和方式2：
        Integer sumSalary2 = personList.stream().reduce(0, (sum, p) -> sum + p.getSalary(), Integer::sum);

        // 求最高工资方式：
        Integer maxSalary = personList.stream().reduce(0, (max, p) -> max > p.getSalary() ? max : p.getSalary(),
                Integer::max);

        System.out.println("工资之和：" + sumSalary.get() + "," + sumSalary2 );
        //输出结果：工资之和：49300,49300
        System.out.println("最高工资：" + maxSalary);
        //输出结果：最高工资：9500
```

### 2.6、收集(collect)

#### 2.6.1 归集(toList/toSet/toMap)

```java
        List<Integer> list = Arrays.asList(1, 6, 3, 4, 6, 7, 9, 6, 20);
        List<Integer> listNew = list.stream().filter(x -> x % 2 == 0).collect(Collectors.toList());
        Set<Integer> set = list.stream().filter(x -> x % 2 == 0).collect(Collectors.toSet());

        List<Person> personList = new ArrayList<Person>();
        personList.add(new Person("Tom", 8900, 23, "male", "New York"));
        personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
        personList.add(new Person("Lily", 7800, 21, "female", "Washington"));
        personList.add(new Person("Anni", 8200, 24, "female", "New York"));

        Map<String, Person> map = personList.stream().filter(p -> p.getSalary() > 8000)
                .collect(Collectors.toMap(Person::getName, p -> p));
        Map<String, Integer> map1 = personList.stream().filter(p -> p.getSalary() > 8000)
                .collect(Collectors.toMap(Person::getName, Person::getAge));
        System.out.println("toList:" + listNew);
        //输出结果：toList：[6, 4, 6, 6, 20]
        System.out.println("toSet:" + set);
        //输出结果：toSet：[4, 20, 6]
        System.out.println("toMap:" + map);
        //输出结果：toMap:{Tom=Person(name=Tom, salary=8900, age=23, sex=male, area=New York), Anni=Person(name=Anni, salary=8200, age=24, sex=female, area=New York)}
        System.out.println("toMap:" + map1);
        ////输出结果：toMap:{Tom=23, Anni=24}
```

#### 2.6.2 统计(count/averaging)

- 计数：`count`
- 平均值：`averagingInt`、`averagingLong`、`averagingDouble`
- 最值：`maxBy`、`minBy`
- 求和：`summingInt`、`summingLong`、`summingDouble`
- 统计以上所有：`summarizingInt`、`summarizingLong`、`summarizingDouble`

> **统计平均工资、工资总额、最高工资**

```java
        List<Person> personList = new ArrayList<Person>();
        personList.add(new Person("Tom", 8900, 23, "male", "New York"));
        personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
        personList.add(new Person("Lily", 7800, 21, "female", "Washington"));


        // 求平均工资
        Double average = personList.stream().collect(Collectors.averagingDouble(Person::getSalary));
        // 求最高工资
        Optional<Integer> max = personList.stream().map(Person::getSalary).max(Integer::compare);
        // 求工资之和
        int sum = personList.stream().mapToInt(Person::getSalary).sum();
        // 一次性统计所有信息
        DoubleSummaryStatistics collect = personList.stream().collect(Collectors.summarizingDouble(Person::getSalary));

        System.out.println("员工平均工资：" + average);
        //输出结果：员工平均工资：7900.0
        System.out.println("员工最高工资：" + sum);
        //输出结果：员工最高工资：23700
        System.out.println("员工工资总和：" + sum);
        //输出结果：员工工资总和：23700
        System.out.println("员工工资所有统计：" + collect);
        //输出结果：员工工资所有统计：DoubleSummaryStatistics{count=3, sum=23700.000000, min=7000.000000, average=7900.000000, max=8900.000000}
```

#### 2.6.3 分组(partitioningBy/groupingBy)

```java
        List<Person> personList = new ArrayList<Person>();
        personList.add(new Person("Tom", 8900, "male", "New York"));
        personList.add(new Person("Jack", 7000, "male", "Washington"));
        personList.add(new Person("Lily", 7800, "female", "Washington"));
        personList.add(new Person("Anni", 8200, "female", "New York"));
        personList.add(new Person("Owen", 9500, "male", "New York"));
        personList.add(new Person("Alisa", 7900,"female", "New York"));

        // 将员工按薪资是否高于8000分组
        Map<Boolean, List<Person>> part = personList.stream().collect(Collectors.partitioningBy(x -> x.getSalary() > 8000));
        // 将员工按性别分组
        Map<String, List<Person>> group = personList.stream().collect(Collectors.groupingBy(Person::getSex));
        // 将员工先按性别分组，再按地区分组
        Map<String, Map<String, List<Person>>> group2 = personList.stream().collect(Collectors.groupingBy(Person::getSex, Collectors.groupingBy(Person::getArea)));
        System.out.println("员工按薪资是否大于8000分组情况：" + part);
        System.out.println("员工按性别分组情况：" + group);
        System.out.println("员工按性别、地区：" + group2);
```

输出结果

```java
员工按薪资是否大于8000分组情况：{false=[Person(name=Jack, salary=7000, age=0, sex=male, area=Washington), Person(name=Lily, salary=7800, age=0, sex=female, area=Washington), Person(name=Alisa, salary=7900, age=0, sex=female, area=New York)], true=[Person(name=Tom, salary=8900, age=0, sex=male, area=New York), Person(name=Anni, salary=8200, age=0, sex=female, area=New York), Person(name=Owen, salary=9500, age=0, sex=male, area=New York)]}
员工按性别分组情况：{female=[Person(name=Lily, salary=7800, age=0, sex=female, area=Washington), Person(name=Anni, salary=8200, age=0, sex=female, area=New York), Person(name=Alisa, salary=7900, age=0, sex=female, area=New York)], male=[Person(name=Tom, salary=8900, age=0, sex=male, area=New York), Person(name=Jack, salary=7000, age=0, sex=male, area=Washington), Person(name=Owen, salary=9500, age=0, sex=male, area=New York)]}
员工按性别、地区：{female={New York=[Person(name=Anni, salary=8200, age=0, sex=female, area=New York), Person(name=Alisa, salary=7900, age=0, sex=female, area=New York)], Washington=[Person(name=Lily, salary=7800, age=0, sex=female, area=Washington)]}, male={New York=[Person(name=Tom, salary=8900, age=0, sex=male, area=New York), Person(name=Owen, salary=9500, age=0, sex=male, area=New York)], Washington=[Person(name=Jack, salary=7000, age=0, sex=male, area=Washington)]}}

```

#### 2.6.4 接合(joining)

```java
        List<Person> personList = new ArrayList<Person>();
        personList.add(new Person("Tom", 8900, 23, "male", "New York"));
        personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
        personList.add(new Person("Lily", 7800, 21, "female", "Washington"));

        String names = personList.stream().map(Person::getName).collect(Collectors.joining(","));
        System.out.println("所有员工的姓名：" + names);
        //输出结果：所有员工的姓名：Tom,Jack,Lily

        List<String> list = Arrays.asList("A", "B", "C");
        String string = list.stream().collect(Collectors.joining("-"));
        System.out.println("拼接后的字符串：" + string);
        //输出结果：拼接后的字符串：A-B-C
```

#### 2.6.5 归约(reducing)

`Collectors`类提供的`reducing`方法，相比于`stream`本身的`reduce`方法，增加了对自定义归约的支持

```java
        List<Person> personList = new ArrayList<Person>();
        personList.add(new Person("Tom", 8900, 23, "male", "New York"));
        personList.add(new Person("Jack", 7000, 25, "male", "Washington"));
        personList.add(new Person("Lily", 7800, 21, "female", "Washington"));

        // 每个员工减去起征点后的薪资之和（这个例子并不严谨，但一时没想到好的例子）
        Integer sum = personList.stream().map(Person::getSalary).reduce(0, (i, j) -> (i + j - 5000));
        System.out.println("员工扣税薪资总和：" + sum);
        //输出结果：员工扣税薪资总和：8700
        // stream的reduce
        Optional<Integer> sum2 = personList.stream().map(Person::getSalary).reduce(Integer::sum);
        System.out.println("员工薪资总和：" + sum2.get());
        //输出结果：员工薪资总和：23700
```

### 2.7、排序(sorted)

- sorted()：自然排序，流中元素需实现Comparable接口
- sorted(Comparator com)：Comparator排序器自定义排序

> **将员工按工资由高到低（工资一样则按年龄由大到小）排序**

```java
        List<Person> personList = new ArrayList<>();

        personList.add(new Person("Sherry", 9000, 24, "female", "New York"));
        personList.add(new Person("Tom", 8900, 22, "male", "Washington"));
        personList.add(new Person("Jack", 9000, 25, "male", "Washington"));
        personList.add(new Person("Lily", 8800, 26, "male", "New York"));
        personList.add(new Person("Alisa", 9000, 26, "female", "New York"));

        // 按工资升序排序（自然排序）
        List<String> newList = personList.stream().sorted(Comparator.comparing(Person::getSalary)).map(Person::getName)
                .collect(Collectors.toList());
        // 按工资倒序排序
        List<String> newList2 = personList.stream().sorted(Comparator.comparing(Person::getSalary).reversed())
                .map(Person::getName).collect(Collectors.toList());
        // 先按工资再按年龄升序排序
        List<String> newList3 = personList.stream()
                .sorted(Comparator.comparing(Person::getSalary).thenComparing(Person::getAge)).map(Person::getName)
                .collect(Collectors.toList());
        // 先按工资再按年龄自定义排序（降序）
        List<String> newList4 = personList.stream().sorted((p1, p2) -> {
            if (p1.getSalary() == p2.getSalary()) {
                return p2.getAge() - p1.getAge();
            } else {
                return p2.getSalary() - p1.getSalary();
            }
        }).map(Person::getName).collect(Collectors.toList());

        System.out.println("按工资升序排序：" + newList);
        //输出结果：按工资升序排序：[Lily, Tom, Sherry, Jack, Alisa]
        System.out.println("按工资降序排序：" + newList2);
        //输出结果：按工资降序排序：[Sherry, Jack, Alisa, Tom, Lily]
        System.out.println("先按工资再按年龄升序排序：" + newList3);
        //输出结果：先按工资再按年龄升序排序：[Lily, Tom, Sherry, Jack, Alisa]
        System.out.println("先按工资再按年龄自定义降序排序：" + newList4);
        //输出结果：先按工资再按年龄自定义降序排序：[Alisa, Jack, Sherry, Tom, Lily]
```

### 2.8、合并/去重/跳过

```java
        String[] arr1 = { "a", "b", "c", "d" };
        String[] arr2 = { "d", "e", "f", "g" };

        Stream<String> stream1 = Stream.of(arr1);
        Stream<String> stream2 = Stream.of(arr2);
        // concat:合并两个流 distinct：去重
        List<String> newList = Stream.concat(stream1, stream2).distinct().collect(Collectors.toList());
        // limit：限制从流中获得前n个数据
        List<Integer> collect = Stream.iterate(1, x -> x + 2).limit(10).collect(Collectors.toList());
        // skip：跳过前n个数据
        List<Integer> collect2 = Stream.iterate(1, x -> x + 2).skip(1).limit(5).collect(Collectors.toList());

        System.out.println("流合并：" + newList);
        //输出结果：流合并：[a, b, c, d, e, f, g]
        System.out.println("limit：" + collect);
        //输出结果：limit：[1, 3, 5, 7, 9, 11, 13, 15, 17, 19]
        System.out.println("skip：" + collect2);
        //输出结果：skip：[3, 5, 7, 9, 11]
```