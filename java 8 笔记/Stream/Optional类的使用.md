# Optional类的使用

## 一、简介

`java.util.Optional`类是Java8为了解决`null`值判断问题，借鉴google guava类库的`Optional`类而引入的一个同名`Optional`类

`java.util.Optional`类可以包含或不包含`null`值的容器对象。如果存在值，则`isPresent`方法将返回`true`，而`get`方法将返回该值

除了`Optional`类之外，还扩展了一些常用类型的`Optional`对象，比如：`OptionalDouble`、`OptionalInt`、`OptionalLong`，用法基本上相似

## 二、Optional类的使用

### 构造方法

```java
@Test
public void testConstructor() {
    // 1、创建一个包装对象值为空的Optional对象
    Optional<String> optStr = Optional.empty();
    // 2、创建包装对象值非空的Optional对象
    Optional<String> optStr1 = Optional.of("optional");
    // 3、创建包装对象值允许为空的Optional对象
    Optional<String> optStr2 = Optional.ofNullable(null);
}
```

### get方法

get()方法主要用于返回包装对象的实际值，但是如果包装对象值为null，会抛出`NoSuchElementException`异常

```java
public T get() {
    if (value == null) {
        throw new NoSuchElementException("No value present");
    }
    return value;
}
```

案例

```java
@Test
public void testGet() {
    Optional<String> optional = Optional.of("yolo");
    Optional<String> optional1 = Optional.ofNullable(null);
    System.out.println(optional.get());
    System.out.println(optional1.get());
}
```

输出结果

```java
yolo

java.util.NoSuchElementException: No value present
```

### isPresent()方法

isPresent()方法用于判断value是否存在，不为NULL则返回true，如果为NULL则返回false

```java
public boolean isPresent() {
    return value != null;
}
```

案例

```java
@Test
public void testIsPresent() {
    Optional<String> optional = Optional.of("yolo");
    Optional<String> optional1 = Optional.ofNullable(null);
    System.out.println(optional.isPresent());
    System.out.println(optional1.isPresent());
}
```

输出结果

```java
true
false
```

### ifPresent(Consumer<? super T> consumer)方法

fPresent()方法接受一个Consumer对象（消费函数），如果包装对象的值非空，运行Consumer对象的accept()方法

```java
public void ifPresent(Consumer<? super T> consumer) {
    if (value != null)
        consumer.accept(value);
}
```

案例

```java
@Test
public void testIfPresent() {
    Optional<String> optional = Optional.of("yolo");
    optional.ifPresent(s -> System.out.println("the String is " + s));
}
```

输出结果

```java
the String is yolo
```

### filter()方法

filter()方法接受参数为Predicate对象，用于对Optional对象进行过滤，如果符合Predicate的条件，返回Optional对象本身，否则返回一个空的Optional对象。

```java
public Optional<T> filter(Predicate<? super T> predicate) {
    Objects.requireNonNull(predicate);
    if (!isPresent()) {
        return this;
    } else {
        return predicate.test(value) ? this : empty();
    }
}
```

案例

```java
@Test
public void testFilter() {
    Optional.of("yolo").filter(s -> s.length() > 2)
            .ifPresent(s -> System.out.println("The length of String is greater than 2 and String is " + s));
}
```

输出结果

```java
The length of String is greater than 2 and String is yolo
```

### map()方法

map()方法的参数为Function（函数式接口）对象，map()方法将Optional中的包装对象用Function函数进行运算，并包装成新的Optional对象（包装对象的类型可能改变）

```java
public <U> Optional<U> map(Function<? super T, ? extends U> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent()) {
        return empty();
    } else {
        return Optional.ofNullable(mapper.apply(value));
    }
}
```

案例

```java
@Test
public void testMap() {
    Optional<String> optional = Optional.of("yolo").map(s -> s.toUpperCase());
    System.out.println(optional.get());
}
```

输出结果

```java
YOLO
```

### flatMap()方法

跟map()方法不同的是，入参Function函数的返回值类型为`Optional<U>`类型，而不是U类型，这样flatMap()能将一个二维的Optional对象映射成一个一维的对象。

```java
public <U> Optional<U> flatMap(Function<? super T, ? extends Optional<? extends U>> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent()) {
        return empty();
    } else {
        @SuppressWarnings("unchecked")
        Optional<U> r = (Optional<U>) mapper.apply(value);
        return Objects.requireNonNull(r);
    }
}
```

案例

```java
@Test
public void testFlatMap() {
    Optional<String> optional = Optional.of("yolo").flatMap(s -> Optional.ofNullable(s.toUpperCase()));
    System.out.println(optional.get());
}
```

输出结果

```java
YOLO
```

### orElse()方法

orElse()方法功能比较简单，即如果包装对象值非空，返回包装对象值，否则返回入参other的值（默认值）

```java
public T orElse(T other) {
    return value != null ? value : other;
}
```

案例

```java
@Test
public void testOrElse() {
    String unkown = (String) Optional.ofNullable(null).orElse("unkown");
    System.out.println(unkown);
}
```

输出结果

```java
unkown
```

### orElseGet()方法

orElseGet()方法与orElse()方法类似，区别在于orElseGet()方法的入参为一个Supplier对象，用Supplier对象的get()方法的返回值作为默认值

```java
public T orElseGet(Supplier<? extends T> supplier) {
    return value != null ? value : supplier.get();
}
```

案例

```java
@Test
public void testOrElseGet() {
    String unkown = (String) Optional.ofNullable(null).orElseGet(() -> "unkown");
    System.out.println(unkown);
}
```

输出结果

```java
unkown
```

### orElseThrow()方法

orElseThrow()方法其实与orElseGet()方法非常相似了，入参都是Supplier对象，只不过orElseThrow()的Supplier对象必须返回一个Throwable异常，并在orElseThrow()中将异常抛出，orElseThrow()方法适用于包装对象值为空时需要抛出特定异常的场景。

```java
public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
    if (value != null) {
        return value;
    } else {
        throw exceptionSupplier.get();
    }
}
```

案例

```java
@Test
public void testOrElseThrow() {
    Optional.ofNullable(null).orElseThrow(() -> new RuntimeException("unkown"));
}
```

输出结果

```java
java.lang.RuntimeException: unkown
```


## 三、总结

- 正确的使用构造方法

- 不确定是否为null时尽量选择`ofNullable`方法
- 通过源代码会发现，它并没有实现`java.io.Serializable`接口，因此应避免在类属性中使用，防止意想不到的问题
- 避免直接调用Optional对象的get和`isPresent`方法，尽量多使用map()、filter()、orElse()等方法来发挥Optional的作用