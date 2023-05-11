# Map集合中put()与putIfAbsent()的区别

put方法：

```
V put(K key, V value);
```

putIfAbsent方法：

```
V putIfAbsent(K key, V value)；
```

我们可以从map官网注释中看出：

1.使用put方法添加键值对，如果map集合中没有该key对应的值，则直接添加，并返回null，<font color='red'>如果已经存在对应的值，则会覆盖旧值，value为新的值。</font>

2.使用putIfAbsent方法添加键值对，如果map集合中没有该key对应的值，则直接添加，并返回null，<font color='red'>如果已经存在对应的值，则依旧为原来的值</font>



### 案例

```java
@Test
    public void test03(){
        Map<String, String> map = new HashMap<>();
        System.out.println(map.put("1", "4"));
        System.out.println(map.get("1"));

        System.out.println(map.put("1", "5"));
        System.out.println(map.get("1"));

        System.out.println(map.putIfAbsent("1", "6"));
        System.out.println(map.get("1"));

        System.out.println(map.putIfAbsent("2","1"));
        System.out.println(map.get("2"));
    }
```

> 输出结果

```
null
4
4
5
5
5
null
1
```