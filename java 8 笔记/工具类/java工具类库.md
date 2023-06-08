## 1.Java自带工具方法

### 1.1 List集合拼接成以逗号分隔的字符串

```java
// 如何把list集合拼接成以逗号分隔的字符串 a,b,c
List<String> list = Arrays.asList("a", "b", "c");
// 第一种方法，可以用stream流
String join = list.stream().collect(Collectors.joining(","));
System.out.println(join); // 输出 a,b,c
// 第二种方法，其实String也有join方法可以实现这个功能
String join = String.join(",", list);
System.out.println(join); // 输出 a,b,c
```

### 1.2 比较两个字符串是否相等，忽略大小写

```java
if (strA.equalsIgnoreCase(strB)) {
  System.out.println("相等");
}
```

### 1.3 比较两个对象是否相等

当我们用equals比较两个对象是否相等的时候，还需要对左边的对象进行判空，不然可能会报空指针异常，我们可以用java.util包下Objects封装好的比较是否相等的方法

```java
Objects.equals(strA, strB);
```

### 1.4 两个List集合取交集

```java
List<String> list1 = new ArrayList<>();
list1.add("a");
list1.add("b");
list1.add("c");
List<String> list2 = new ArrayList<>();
list2.add("a");
list2.add("b");
list2.add("d");
list1.retainAll(list2);
System.out.println(list1); // 输出[a, b]
```

## 2.apache commons工具类库

### 2.1 commons-lang，java.lang的增强版

> maven依赖是：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

#### 2.1.1 字符串判空

```java
public static boolean isEmpty(final CharSequence cs) {
    return cs == null || cs.length() == 0;
}

public static boolean isNotEmpty(final CharSequence cs) {
    return !isEmpty(cs);
}

// 判空的时候，会去除字符串中的空白字符，比如空格、换行、制表符
public static boolean isBlank(final CharSequence cs) {
    final int strLen = length(cs);
    if (strLen == 0) {
        return true;
    }
    for (int i = 0; i < strLen; i++) {
        if (!Character.isWhitespace(cs.charAt(i))) {
            return false;
        }
    }
    return true;
}

public static boolean isNotBlank(final CharSequence cs) {
    return !isBlank(cs);
}
```

#### 2.1.2 首字母转成大写

```java
String str = "yideng";
String capitalize = StringUtils.capitalize(str);
System.out.println(capitalize); // 输出Yideng
```

#### 2.1.3 重复拼接字符串

```java
String str = StringUtils.repeat("ab", 2);
System.out.println(str); // 输出abab
```

#### 2.1.4 格式化日期

```java
// Date类型转String类型
String date = DateFormatUtils.format(new Date(), "yyyy-MM-dd HH:mm:ss");
System.out.println(date); // 输出 2021-05-01 01:01:01

// String类型转Date类型
Date date = DateUtils.parseDate("2021-05-01 01:01:01", "yyyy-MM-dd HH:mm:ss");

// 计算一个小时后的日期
Date date = DateUtils.addHours(new Date(), 1);
```

#### 2.1.5 包装临时对象

```java
// 返回两个字段
ImmutablePair<Integer, String> pair = ImmutablePair.of(1, "yideng");
System.out.println(pair.getLeft() + "," + pair.getRight()); // 输出 1,yideng
// 返回三个字段
ImmutableTriple<Integer, String, Date> triple = ImmutableTriple.of(1, "yideng", new Date());
System.out.println(triple.getLeft() + "," + triple.getMiddle() + "," + triple.getRight()); // 输出 1,yideng,Wed Apr 07 23:30:00 CST 2021
```

### 2.2 commons-collections 集合工具类

> Maven依赖是：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
</dependency>
```

#### 2.2.1 集合判空

封装了集合判空的方法，以下是源码：

```java
public static boolean isEmpty(final Collection<?> coll) {
    return coll == null || coll.isEmpty();
}

public static boolean isNotEmpty(final Collection<?> coll) {
    return !isEmpty(coll);
}
// 两个集合取交集
Collection<String> collection = CollectionUtils.retainAll(listA, listB);
// 两个集合取并集
Collection<String> collection = CollectionUtils.union(listA, listB);
// 两个集合取差集
Collection<String> collection = CollectionUtils.subtract(listA, listB);
```

### 2.3 common-beanutils 操作对象

> Maven依赖：

```xml
<dependency>
    <groupId>commons-beanutils</groupId>
    <artifactId>commons-beanutils</artifactId>
    <version>1.9.4</version>
</dependency>
```

> 实体类：

```java
  public class User {
        private Integer id;
        private String name;
        }
```

> 设置对象属性：

```java
User user = new User();
BeanUtils.setProperty(user, "id", 1);
BeanUtils.setProperty(user, "name", "yideng");
System.out.println(BeanUtils.getProperty(user, "name")); // 输出 yideng
System.out.println(user); // 输出 {"id":1,"name":"yideng"}
```

> 对象和map互转

```java
// 对象转map
Map<String, String> map = BeanUtils.describe(user);
System.out.println(map); // 输出 {"id":"1","name":"yideng"}
// map转对象
User newUser = new User();
BeanUtils.populate(newUser, map);
System.out.println(newUser); // 输出 {"id":1,"name":"yideng"}
```

### 2.4 commons-io 文件流处理

> Maven依赖：

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.8.0</version>
</dependency>
```

> 文件处理

```java
File file = new File("demo1.txt");
// 读取文件
List<String> lines = FileUtils.readLines(file, Charset.defaultCharset());
// 写入文件
FileUtils.writeLines(new File("demo2.txt"), lines);
// 复制文件
FileUtils.copyFile(srcFile, destFile);
```

## 3.Google Guava 工具类库

> Maven依赖：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>30.1.1-jre</version>
</dependency>
```

### 3.1 创建集合

```java
List<String> list = Lists.newArrayList();
List<Integer> list = Lists.newArrayList(1, 2, 3);
// 反转list
List<Integer> reverse = Lists.reverse(list);
System.out.println(reverse); // 输出 [3, 2, 1]
// list集合元素太多，可以分成若干个集合，每个集合10个元素
List<List<Integer>> partition = Lists.partition(list, 10);

Map<String, String> map = Maps.newHashMap();
Set<String> set = Sets.newHashSet();
```

### 3.2 黑科技集合

#### 3.2.1 Multimap 一个key可以映射多个value的HashMap

```java
Multimap<String, Integer> map = ArrayListMultimap.create();
map.put("key", 1);
map.put("key", 2);
Collection<Integer> values = map.get("key");
System.out.println(map); // 输出 {"key":[1,2]}
// 还能返回你以前使用的臃肿的Map
Map<String, Collection<Integer>> collectionMap = map.asMap();
```

#### 3.2.2 BiMap 一种连value也不能重复的HashMap

```java
BiMap<String, String> biMap = HashBiMap.create();
// 如果value重复，put方法会抛异常，除非用forcePut方法
biMap.put("key","value");
System.out.println(biMap); // 输出 {"key":"value"}
// 既然value不能重复，何不实现个翻转key/value的方法，已经有了
BiMap<String, String> inverse = biMap.inverse();
System.out.println(inverse); // 输出 {"value":"key"}
```

#### 3.2.3 Table 一种有两个key的HashMap

```java
// 一批用户，同时按年龄和性别分组
Table<Integer, String, String> table = HashBasedTable.create();
table.put(18, "男", "yideng");
table.put(18, "女", "Lily");
System.out.println(table.get(18, "男")); // 输出 yideng
// 这其实是一个二维的Map，可以查看行数据
Map<String, String> row = table.row(18);
System.out.println(row); // 输出 {"男":"yideng","女":"Lily"}
// 查看列数据
Map<Integer, String> column = table.column("男");
System.out.println(column); // 输出 {18:"yideng"}
```

#### 3.2.4 Multiset 一种用来计数的Set

```java
Multiset<String> multiset = HashMultiset.create();
multiset.add("apple");
multiset.add("apple");
multiset.add("orange");
System.out.println(multiset.count("apple")); // 输出 2
// 查看去重的元素
Set<String> set = multiset.elementSet();
System.out.println(set); // 输出 ["orange","apple"]
// 还能查看没有去重的元素
Iterator<String> iterator = multiset.iterator();
while (iterator.hasNext()) {
    System.out.println(iterator.next());
}
// 还能手动设置某个元素出现的次数
multiset.setCount("apple", 5);
```

## 4.其他工具类库

### org.apache.commons.io.IOUtils

```java
closeQuietly：关闭一个IO流、socket、或者selector且不抛出异常，通常放在finally块
toString：转换IO流、 Uri、 byte[]为String
copy：IO流数据复制，从输入流写到输出流中，最大支持2GB
toByteArray：从输入流、URI获取byte[]
write：把字节. 字符等写入输出流
toInputStream：把字符转换为输入流
readLines：从输入流中读取多行数据，返回List<String>
copyLarge：同copy，支持2GB以上数据的复制
lineIterator：从输入流返回一个迭代器，根据参数要求读取的数据量，全部读取，如果数据不够，则失败
```

### org.apache.commons.io.FileUtils

```java
deleteDirectory：删除文件夹
readFileToString：以字符形式读取文件内容
deleteQueitly：删除文件或文件夹且不会抛出异常
copyFile：复制文件
writeStringToFile：把字符写到目标文件，如果文件不存在，则创建
forceMkdir：强制创建文件夹，如果该文件夹父级目录不存在，则创建父级
write：把字符写到指定文件中
listFiles：列举某个目录下的文件(根据过滤器)
copyDirectory：复制文件夹
forceDelete：强制删除文件
```

### org.apache.commons.lang.StringUtils

```java
isBlank：字符串是否为空 (trim后判断)
isEmpty：字符串是否为空 (不trim并判断)
equals：字符串是否相等
join：合并数组为单一字符串，可传分隔符
split：分割字符串
EMPTY：返回空字符串
trimToNull：trim后为空字符串则转换为null
replace：替换字符串
```

### org.apache.http.util.EntityUtils

```java
toString：把Entity转换为字符串
consume：确保Entity中的内容全部被消费。可以看到源码里又一次消费了Entity的内容，假如用户没有消费，那调用Entity时候将会把它消费掉
toByteArray：把Entity转换为字节流
consumeQuietly：和consume一样，但不抛异常
getContentCharset：获取内容的编码
```

### org.apache.commons.lang3.StringUtils

```java
isBlank：字符串是否为空 (trim后判断)
isEmpty：字符串是否为空 (不trim并判断)
equals：字符串是否相等
join：合并数组为单一字符串，可传分隔符
split：分割字符串
EMPTY：返回空字符串
replace：替换字符串
capitalize：首字符大写
```

### org.apache.commons.io.FilenameUtils

```java
getExtension：返回文件后缀名
getBaseName：返回文件名，不包含后缀名
getName：返回文件全名
concat：按命令行风格组合文件路径(详见方法注释)
removeExtension：删除后缀名
normalize：使路径正常化
wildcardMatch：匹配通配符
seperatorToUnix：路径分隔符改成unix系统格式的，即/
getFullPath：获取文件路径，不包括文件名
isExtension：检查文件后缀名是不是传入参数(List<String>)中的一个

```

### org.springframework.util.StringUtils

```java
hasText：检查字符串中是否包含文本
hasLength：检测字符串是否长度大于0
isEmpty：检测字符串是否为空（若传入为对象，则判断对象是否为null）
commaDelimitedStringToArray：逗号分隔的String转换为数组
collectionToDelimitedString：把集合转为CSV格式字符串
replace 替换字符串
delimitedListToStringArray：相当于split
uncapitalize：首字母小写
collectionToDelimitedCommaString：把集合转为CSV格式字符串
tokenizeToStringArray：和split基本一样，但能自动去掉空白的单词
```

### org.apache.commons.lang.ArrayUtils

```java
contains：是否包含某字符串
addAll：添加整个数组
clone：克隆一个数组
isEmpty：是否空数组
add：向数组添加元素
subarray：截取数组
indexOf：查找某个元素的下标
isEquals：比较数组是否相等
toObject：基础类型数据数组转换为对应的Object数组
```

### org.apache.http.client.utils.URLEncodedUtils

```java
format：格式化参数，返回一个HTTP POST或者HTTP PUT可用application/x-www-form-urlencoded字符串
parse：把String或者URI等转换为List<NameValuePair>
```

### org.apache.commons.codec.digest.DigestUtils

```java
md5Hex：MD5加密，返回32位字符串
sha1Hex：SHA-1加密
sha256Hex：SHA-256加密
sha512Hex：SHA-512加密
md5：MD5加密，返回16位字符串
```

### org.apache.commons.collections.CollectionUtils

```java
isEmpty：是否为空
select：根据条件筛选集合元素
transform：根据指定方法处理集合元素，类似List的map()
filter：过滤元素，雷瑟List的filter()
find：基本和select一样
collect：和transform 差不多一样，但是返回新数组
forAllDo：调用每个元素的指定方法
isEqualCollection：判断两个集合是否一致
```

### org.apache.commons.lang3.ArrayUtils

```java
contains：是否包含某个字符串
addAll：添加整个数组
clone：克隆一个数组
isEmpty：是否空数组
add：向数组添加元素
subarray：截取数组
indexOf：查找某个元素的下标
isEquals：比较数组是否相等
toObject：基础类型数据数组转换为对应的Object数组

```

### org.apache.commons.beanutils.PropertyUtils

```java
getProperty：获取对象属性值
setProperty：设置对象属性值
getPropertyDiscriptor：获取属性描述器
isReadable：检查属性是否可访问
copyProperties：复制属性值，从一个对象到另一个对象
getPropertyDiscriptors：获取所有属性描述器
isWriteable：检查属性是否可写
getPropertyType：获取对象属性类型
```

### org.apache.commons.beanutils.BeanUtils

```java
copyPeoperties：复制属性值，从一个对象到另一个对象
getProperty：获取对象属性值
setProperty：设置对象属性值
populate：根据Map给属性复制
copyPeoperty：复制单个值，从一个对象到另一个对象
cloneBean：克隆bean实例
```

> 另外，工具类，根据阿里开发手册，包名如果要使用util不能带s，工具类命名为 XxxUtils