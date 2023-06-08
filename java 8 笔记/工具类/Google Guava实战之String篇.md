# Google Guava 实战之String篇
## 一、导入依赖

```xml
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>27.0.1-jre</version>
        </dependency>
```

## 二、String

### 字符串分割

```java
 	@Test
    public void test(){
        //Splitter.on(char):按单个字符拆分  Splitter.on(‘,’)
        Splitter splitter = Splitter.on(",");
        String str = "foo,bar,,   qux";
        String str1 = "hello,world!guava!!";
        String str2 = "fo??o??????bar?????qux????";
        String str3 = "abcd1234efg";
        String str4 = "ABCd1234Efg";
        String str5 = "ABCd1234./]?*@#$(*&Efg";
        String str6 = "ab34cd154ef567g";
        String str7 = "ab5yt5cd55e0fg55hi123jkl4mno65pq9x9yz7vtw";
        String str8 = "abcd11abcd22abcd33abcdaba";
        String str9 = "   a 2b 3c 44 de fh k  % %$ # ";

        //1.直接分割
        Iterable<String> split = splitter.split(str);
        System.out.println("1==============split:" + split);
        //2.trimResults():自动删除开头和结尾的空白 ，从每个返回字符串
        Iterable<String> split2 = splitter.trimResults().split(str);
        System.out.println("2==============split2:" + split2);
        //3.omitEmptyStrings():自动省略从结果空字符串
        Iterable<String> split3 = splitter.omitEmptyStrings().split(str);
        System.out.println("3==============split3:" + split3);
        //4、trimResults+omitEmptyStrings:去空格+去空字符串
        Iterable<String> split4 = splitter.omitEmptyStrings().trimResults().split(str);
        System.out.println("4==============split4:" + split4);
        //5、splitToList(string):将字符串分割，返回一个list
        //5.1
        List<String> list = splitter.splitToList(str);
        System.out.println("5.1=============" + list);
        //5.2
        List<String> list1 = splitter.omitEmptyStrings().trimResults().splitToList(str);
        System.out.println("5.2=============" + list1);
        //6、fixedLength：字符串按固定长度分割
        Iterable<String> fixedSplit = Splitter.fixedLength(3).omitEmptyStrings().trimResults().split(str);
        System.out.println("6==============fixedSplit:" + fixedSplit);
        //7.根据正则分割
        Splitter splitterPattern = Splitter.onPattern("\\?").omitEmptyStrings().trimResults();
        Iterable<String> split1 = splitterPattern.split(str2);
        System.out.println("7==============" + split1);
    }
```

> 输出结果

```java
1==============split:[foo, bar, ,    qux]
2==============split2:[foo, bar, , qux]
3==============split3:[foo, bar,    qux]
4==============split4:[foo, bar, qux]
5.1=============[foo, bar, ,    qux]
5.2=============[foo, bar, qux]
6==============fixedSplit:[foo, ,ba, r,,, qux]
7==============[fo, o, bar, qux]    
```

### 字符串截取

```java
    @Test
    public void test(){
        //Splitter.on(char):按单个字符拆分  Splitter.on(‘,’)
        Splitter splitter = Splitter.on(",");
        String str = "foo,bar,,   qux";
        String str1 = "hello,world!guava!!";
        String str2 = "fo??o??????bar?????qux????";
        String str3 = "abcd1234efg";
        String str4 = "ABCd1234Efg";
        String str5 = "ABCd1234./]?*@#$(*&Efg";
        String str6 = "ab34cd154ef567g";
        String str7 = "ab5yt5cd55e0fg55hi123jkl4mno65pq9x9yz7vtw";
        String str8 = "abcd11abcd22abcd33abcdaba";
        String str9 = "   a 2b 3c 44 de fh k  % %$ # ";

        //8.字符匹配器
        //8.1 CharMatcher.javaLetter() 匹配字母  (已废弃)
        String removeFrom = CharMatcher.javaLetter().removeFrom(str3);
        System.out.println("8.1==============CharMatcher.javaLetter():" + removeFrom);
        //8.2 CharMatcher.javaUpperCase() 匹配大写字母 (已废弃)
        CharMatcher charMatcher = CharMatcher.javaUpperCase();
        String upperCaseStr = charMatcher.removeFrom(str4);
        System.out.println("8.2===============upperCaseStr:" + upperCaseStr);
        //8.3 CharMatcher.javaUpperCase() 匹配大写字母 (已废弃)
        CharMatcher lowerCaseCharMatcher = CharMatcher.javaLowerCase();
        String lowerCaseStr = lowerCaseCharMatcher.removeFrom(str4);
        System.out.println("8.3===============lowerCaseStr:" + lowerCaseStr);
        //8.4 CharMatcher.digit() 匹配ASCII数字  (已废弃)
        //CharMatcher.digit().retainFrom(str) 只保留数字
        String retainFrom = CharMatcher.digit().retainFrom(str5);
        System.out.println("8.4===============retainFrom:" + retainFrom);
        //8.5 CharMatcher.javaDigit() 匹配UNICODE数字  (已废弃)
        //用*号替换所有数字
        String replaceFrom = CharMatcher.javaDigit().replaceFrom(str6, "*");
        System.out.println("8.5===============replaceFrom:" + replaceFrom);
        //8.6 CharMatcher.anyOf() 表明你想匹配的所有字符
        CharMatcher abcdCharMatcher = CharMatcher.anyOf("abcd");
        String abcdStr = abcdCharMatcher.removeFrom(str3);
        System.out.println("8.6======================" + abcdStr);
        //8.7 CharMatcher.is():表明你想匹配的一个确定的字符
        CharMatcher charMatcher1 = CharMatcher.is('a');
        String isaStr = charMatcher1.removeFrom(str3);
        System.out.println("8.7======================" + isaStr);
        //8.8 表明你想匹配的一个字符范围，例如：CharMatcher.inRange('a', 'z')
        CharMatcher inRange = CharMatcher.inRange('a', 'e');
        String inRangeStr = inRange.removeFrom(str7);
        System.out.println("8.8======================" + inRangeStr);
        //8.9去除所有空格
        String s = CharMatcher.breakingWhitespace().removeFrom(str9);
        System.out.println("8.9===============CharMatcher.breakingWhitespace():" + s);
        String s1 = CharMatcher.whitespace().removeFrom(str9);
        System.out.println("8.9===============CharMatcher.whitespace():" + s1);
        //9.collapseFrom 把每组连续的匹配字符替换为特定字符
        String s2 = CharMatcher.anyOf("eko").collapseFrom("bookkeeper", '-');
        System.out.println("9============collapseFrom==========" + s2);
        //10 matchesAllOf 测试是否字符序列中的所有字符都匹配
        boolean b = CharMatcher.anyOf("eko").matchesAllOf("bookkeeper");
        System.out.println("10.1========matchesAllOf============" + b);
        boolean b2 = CharMatcher.anyOf("eko").matchesAllOf("ekookoeek");
        System.out.println("10.2========matchesAllOf============" + b2);
        //11 trimFrom 移除字符序列的前导匹配字符和尾部匹配字符。
        String s3 = CharMatcher.anyOf("ab").trimFrom(str8);
        System.out.println("11===========trimFrom=========" + s3);
        //12 removeFrom(CharSequence)
        //从字符序列中移除所有匹配字符。
        String s4 = CharMatcher.anyOf("ab").removeFrom(str8);
        System.out.println("12===========removeFrom=========" + s4);
        //13 retainFrom(CharSequence)
        //在字符序列中保留匹配字符，移除其他字符。
        String s5 = CharMatcher.anyOf("ab").retainFrom(str8);
        System.out.println("13===========retainFrom=========" + s5);
    }
```

> 输出结果

```java
8.1==============CharMatcher.javaLetter():1234
8.2===============upperCaseStr:d1234fg
8.3===============lowerCaseStr:ABC1234E
8.4===============retainFrom:1234
8.5===============replaceFrom:ab**cd***ef***g
8.6======================1234efg
8.7======================bcd1234efg
8.8======================5yt5550fg55hi123jkl4mno65pq9x9yz7vtw
8.9===============CharMatcher.breakingWhitespace():a2b3c44defhk%%$#
8.9===============CharMatcher.whitespace():a2b3c44defhk%%$#
9============collapseFrom==========b-p-r
10.1========matchesAllOf============false
10.2========matchesAllOf============true
11===========trimFrom=========cd11abcd22abcd33abcd
12===========removeFrom=========cd11cd22cd33cd
13===========retainFrom=========abababababa
```

### 字符串格式大小转换

```java
    @Test
    public void test(){
        //Splitter.on(char):按单个字符拆分  Splitter.on(‘,’)
        Splitter splitter = Splitter.on(",");
        String str = "foo,bar,,   qux";
        String str1 = "hello,world!guava!!";
        String str2 = "fo??o??????bar?????qux????";
        String str3 = "abcd1234efg";
        String str4 = "ABCd1234Efg";
        String str5 = "ABCd1234./]?*@#$(*&Efg";
        String str6 = "ab34cd154ef567g";
        String str7 = "ab5yt5cd55e0fg55hi123jkl4mno65pq9x9yz7vtw";
        String str8 = "abcd11abcd22abcd33abcdaba";
        String str9 = "   a 2b 3c 44 de fh k  % %$ # ";

        /* 14、
           字符串格式大小转换
            格式:                范例:
            LOWER_CAMEL	        lowerCamel
            LOWER_HYPHEN	    lower-hyphen
            LOWER_UNDERSCORE	lower_underscore
            UPPER_CAMEL	        UpperCamel
            UPPER_UNDERSCORE	UPPER_UNDERSCORE
         */
        String to = CaseFormat.UPPER_UNDERSCORE.to(CaseFormat.LOWER_CAMEL, "dAtE_DaTe");
        System.out.println(to);
        String to5 = CaseFormat.UPPER_UNDERSCORE.to(CaseFormat.UPPER_CAMEL, "dAtE_DaTe");
        System.out.println(to5);
        String to1 = CaseFormat.LOWER_CAMEL.to(CaseFormat.LOWER_CAMEL, "dAtE_DaTe");
        System.out.println(to1);
        String to2 = CaseFormat.LOWER_HYPHEN.to(CaseFormat.LOWER_CAMEL, "dAtE_DaTe");
        System.out.println(to2);
        String to3 = CaseFormat.LOWER_UNDERSCORE.to(CaseFormat.LOWER_CAMEL, "dAt_EDaTe");
        System.out.println(to3);
        String to4 = CaseFormat.UPPER_CAMEL.to(CaseFormat.LOWER_CAMEL, "dAtEDaTe");
        System.out.println(to4);
    }
```

> 输出结果

```java
dateDate
DateDate
dAtE_DaTe
date_date
datEdate
dAtEDaTe
```

### 统计所匹配字符

```java
    @Test
    public void test(){
        //Splitter.on(char):按单个字符拆分  Splitter.on(‘,’)
        Splitter splitter = Splitter.on(",");
        String str = "foo,bar,,   qux";
        String str1 = "hello,world!guava!!";
        String str2 = "fo??o??????bar?????qux????";
        String str3 = "abcd1234efg";
        String str4 = "ABCd1234Efg";
        String str5 = "ABCd1234./]?*@#$(*&Efg";
        String str6 = "ab34cd154ef567g";
        String str7 = "ab5yt5cd55e0fg55hi123jkl4mno65pq9x9yz7vtw";
        String str8 = "abcd11abcd22abcd33abcdaba";
        String str9 = "   a 2b 3c 44 de fh k  % %$ # ";
//15 统计所匹配字符
        //次数
        int aCount = CharMatcher.is('a').countIn(str8);
        System.out.println(aCount);
        //第一次匹配到的下标
        int aIndex = CharMatcher.is('b').indexIn(str8);
        System.out.println(aIndex);
        //最后一次匹配到的下标
        int lastIndexIn = CharMatcher.is('b').lastIndexIn(str8);
        System.out.println(lastIndexIn);
    }
```

> 输出结果

```java
6
1
23
```

### Joiner 连接器

```java
	@Test
    public void test2(){
        /**
         * 1、Joiner 连接器
         * 用分隔符把字符串序列连接起来也可能会遇上不必要的麻烦。如果字符串序列中含有null，那连接操作会更难，就可以使用Joiner
         * 如果既未指定也skipNulls()未useForNull(String)指定,使用null,会报NullPointerException
         *
         * skipNulls():就是跳过null
         * useForNull(String)：就是如果有null,则换成你给的默认值
         */
        Joiner joiner = Joiner.on(";").useForNull("我是null");
        String join = joiner.join("hello", null, "world", "guava");
        System.out.println("Joiner=============使用useForNull=========：" + join);

        Joiner joiner2 = Joiner.on(";").skipNulls();
        String join2 = joiner2.join("hello", null, "world", "guava");
        System.out.println("Joiner=============使用skipNulls========：" + join2);

        StringBuilder stringBuilder = new StringBuilder();
        Joiner joiner3 = Joiner.on(":").skipNulls();
        StringBuilder sb = joiner3.appendTo(stringBuilder, "hello", null, "world", "guava");
        System.out.println("Joiner.appendTo=============使用skipNulls========：" + sb);

        /**
         *  withKeyValueSeparator(char keyValueSeparator)
         *  withKeyValueSeparator 方法指定了键与值的分隔符，同时返回一个 MapJoiner 实例。
         *  Map里可以插入键或值为空指针的键值对，如果我们要拼接这种 Map，千万记得要用 useForNull对 MapJoiner做保护
         *  ps：不能使用skipNulls()跳过null，否者Exception in thread "main" java.lang.UnsupportedOperationException: can't use .skipNulls() with maps
         *  这里可以使用useForNull(string)对null值和null键进行处理
         */
        Map<String, Object> map = new HashMap<>();
        map.put("张三", 123);
        map.put("李四", 457);
        map.put("王五", 235);
        map.put("马六", 752);
        map.put("田七", null);
        map.put(null, 000);
        Joiner joiner4 = Joiner.on(":").useForNull("我是null");
        Joiner.MapJoiner mapJoiner = joiner4.withKeyValueSeparator("=>");
        String joinMap = mapJoiner.join(map);
        System.out.println("Joiner=============withKeyValueSeparator==========="+joinMap);
    }
```

> 输出结果

```java
Joiner=============使用useForNull=========：hello;我是null;world;guava
Joiner=============使用skipNulls========：hello;world;guava
Joiner.appendTo=============使用skipNulls========：hello:world:guava
Joiner=============withKeyValueSeparator===========我是null=>0:李四=>457:张三=>123:马六=>752:王五=>235:田七=>我是null
```