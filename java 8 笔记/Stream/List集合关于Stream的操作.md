# List集合关于Stream的操作
### 1.String[ ] 转 List< String>

```java
        String[] array = new String[]{"aa","bb","cc"};
        List<String> strList = Arrays.stream(array).collect(Collectors.toList());
```

### 2.String[ ] 转 Integer [ ]

```java
        String[] strArry = new String[]{"5", "6", "1", "4", "9"};
        Integer[] integerArry = Arrays.stream(strArry).map(Integer::parseInt).toArray(Integer[]::new);
```

### 3.int [ ] 转 List< Integer>

```java
        int[] intArry = new int[]{5,6,1,4,9};
        List<Integer> integerList = Arrays.stream(intArry).boxed().collect(Collectors.toList());
```

### 4.int [ ] 转 Integer [ ]

```java
        int[] intArry = new int[]{5, 6, 1, 4, 9};
        Integer[] integerArry = Arrays.stream(intArry).boxed().toArray(Integer[]::new);
```

### 5.List< String> 逗号拼接为一个字符串

```java
        List<String> strList = new ArrayList<>(Arrays.asList("a","b","c"));
        String str = strList.stream().collect(Collectors.joining(","));
        System.out.println(str);//a,b,c
```

### 6.List< Integer> 逗号拼接为一个字符串

```java
        List<Integer> integerList = Arrays.asList(7, 8, 9);
        String str = integerList.stream().map(v->String.valueOf(v)).collect(Collectors.joining(","));
        System.out.println(str);//7,8,9
```

### 7.判断数组中是否含有某一值

```java
        //字符串数组
		String[] values = {"AB","BC","CD","AE"};
        boolean contains = Arrays.stream(values).anyMatch("AE"::equals);
		//int数组
        int[] a = {1,2,3,4};
        boolean contains = IntStream.of(a).anyMatch(x -> x == 4);
```

### 8.数组或List求和

```java
        //数组求和
		int[] intArray = {11, 5, 3, 2, 1};
        int sum = Arrays.stream(intArray).reduce(0, Integer::sum);
		//List求和
        List<Integer> list = new ArrayList<>(Arrays.asList(5, 1, 7, 10));
        int sum = list.stream().reduce(0, Integer::sum);
```

### 9.两个数组合并为一个新的数组

```java
		//两个字符串数组合并为一个新的数组
        String[] a = {"a", "b", "c"};
        String[] b = {"1", "2", "3"};
        String[] c = Stream.of(a,b).flatMap(Stream::of).toArray(String[]::new);

		//两个 int 型数组合并为一个新的 int 型数组
        int[] a = new int[]{1,3};
        int[] b = new int[]{2,4};
        int[] c =  IntStream.concat(Arrays.stream(a), Arrays.stream(b)).toArray();
```

### 10.两个数组合并为一个List

```java
		//两个String数组转List<String>
        String[] a = {"a", "b", "c"};
        String[] b = {"1", "2", "3"};
        List<String> strList = Stream.of(a, b).flatMap(Stream::of).collect(Collectors.toList());

		//两个int数组转List<Integer>
        int[] a = new int[]{1, 2};
        int[] b = new int[]{3, 4};
        List<Integer> integerList = Stream.of(IntStream.of(a).boxed(), IntStream.of(b).boxed()).flatMap(s -> s).collect(Collectors.toList());
```

### 11.两个List合并为一个新的List

```java
        List<String> lis1 = new ArrayList<>(Arrays.asList("a", "b", "c"));
        List<String> list2 = new ArrayList<>(Arrays.asList("e", "f", "g"));
        List<String> newList = Stream.of(lis1, list2).flatMap(Collection::stream).collect(Collectors.toList());
```

### 12.List< Integer>求交集、并集、差集

```java
List<Integer> list = new ArrayList<>(Arrays.asList(7, 8, 9));
List<Integer> list2 = new ArrayList<>(Arrays.asList(3,4, 9));

//交集
 List<Integer> beMixed = list.stream().filter(list2::contains).collect(Collectors.toList());
 System.out.println(beMixed);//[9]

//并集
List<Integer> aggregate = Stream.of(list, list2).flatMap(Collection::stream).distinct().collect(Collectors.toList());
System.out.println(aggregate);//[7, 8, 9, 3, 4]

//差集
List<Integer> subtraction = list.stream().filter(v->!list2.contains(v)).collect(Collectors.toList());
System.out.println(subtraction);//[7, 8]
```

### 13.Map的value保存为List

```java
        Map<String, String> map = new HashMap<>();
        map.put("1", "a");
        map.put("2", "b");
        map.put("3", "c");
        List<String> values = map.values().stream().collect(Collectors.toList());
```

### 14.Map<String, List< Integer>>获取所有的values为一个List< Integer>

```java
        List<Integer> list1 = new ArrayList<>(Arrays.asList(1,2,3));
        List<Integer> list2 = new ArrayList<>(Arrays.asList(4, 5, 6));
        List<Integer> list3 = new ArrayList<>(Arrays.asList(7, 8, 9));

        Map<String, List<Integer>> map = new HashMap<>();
        map.put("a", list1);
        map.put("b", list2);
        map.put("c", list3);

        List<Integer> list = map.entrySet().stream().map(e -> e.getValue()).flatMap(Collection::stream).collect(Collectors.toList());
        System.out.println(list);//[a, b, c, d, e, f, g, h, i]

```

### 15.List< String>统计各字符串出现的次数

```java
        List<String> items = Arrays.asList("apple", "apple", "orange", "orange", "orange", "blueberry", "peach", "peach", "peach", "peach");
        Map<String, Long> result = items.stream().collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
        //{orange=3, apple=2, blueberry=1, peach=4}
        System.out.println(result);
```

### 16.List<List< String>>转List< String>

```java
        List<String> a = Arrays.asList("Virat", "Dhoni", "Jadeja");
        List<String> b = Arrays.asList("1", "2", "3");
        List<List<String>> list = new ArrayList<>();
        list.add(a);
        list.add(b);

        List<String> newList = list.stream().flatMap(v -> v.stream()).collect(Collectors.toList());
        //[Virat, Dhoni, Jadeja, 1, 2, 3]
        System.out.println(newList);
```

### 17.List< Object>转Map<String, Object>

```java
        List<Student> list = new ArrayList<>();
        list.add(new Student(1, "张三", 85));
        list.add(new Student(2, "李四", 60));
        Map<String, Student> map = list.stream().collect(Collectors.toMap(Student::getName, Function.identity()));
```

### 18.List< Object>针对某一成员变量获取最大、小值的Object

```java
        List<Student> list = new ArrayList<>();
        list.add(new Student(1, "张三", 85));
        list.add(new Student(2, "李四", 60));
        list.add(new Student(3, "刘一", 70));
        list.add(new Student(4, "李四", 99));
        //获取score最大值
        Optional<Student> maxOptStudent = list.stream().collect(Collectors.maxBy(Comparator.comparing(Student::getScore)));
        Student maxStudent = maxOptStudent.get();
        //获取score最小值
        Optional<Student> mixOptStudent = list.stream().collect(Collectors.minBy(Comparator.comparing(Student::getScore)));
        Student minxStudent = mixOptStudent.get();

```

### 19.List< Object>获取某个成员变量最大、最小值、平均值，求和

```java
        List<Student> list = new ArrayList<>();
        list.add(new Student(1, "小名", 17));
        list.add(new Student(2, "小红", 18));
        list.add(new Student(3, "小蓝", 19));
        list.add(new Student(4, "小灰", 20));
        list.add(new Student(5, "小黄", 21));
        list.add(new Student(6, "小白", 22));
        
        IntSummaryStatistics intSummary = list.stream().collect(Collectors.summarizingInt(Student::getScore));
        System.out.println(intSummary.getAverage());// 19.5
        System.out.println(intSummary.getMax());// 22
        System.out.println(intSummary.getMin());// 17
        System.out.println(intSummary.getSum());// 117

```

### 20.List 获取最大、最小值、平均值，求和

```java
        List<Integer> list = new ArrayList<>(Arrays.asList(12,3,34,67,100,99));
        IntSummaryStatistics statistics = list.stream().mapToInt(value -> value).summaryStatistics();
        System.out.println("the max:" + statistics.getMax());
        System.out.println("the min:" + statistics.getMin());
        System.out.println("the average:" + statistics.getAverage());
        System.out.println("the sum:" + statistics.getSum());
        System.out.println("the count:" + statistics.getCount());

```