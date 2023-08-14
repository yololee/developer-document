# easyexcel常见注解

## 一、依赖

```xml
        <!--阿里巴巴EasyExcel依赖-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>easyexcel</artifactId>
            <version>3.3.1</version>
        </dependency>
```

> 注解所在位置

![image-20230814095045718](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230814095045718.png)


## 二、常见注解

### @ExcelProperty

注解中有三个参数`value`**,**`index`**,**`converter`分别代表列明，列序号，数据转换方式

```java
public class ExportModel {
    @ExcelProperty({"制造商"})
    private String manufacturer;

    @ExcelProperty({"型号"})
    private String model;
}
```

### @ColumnWith(设置列宽)

设置列宽度的注解,注解中只有一个参数value，value的单位是字符长度，最大可以设置255个字符

```java
public class ExportModel {
    @ColumnWidth(30)
    private String manufacturer;

    @ColumnWidth(30)
    private String model;
}
```

### @ContentFontStyle(字体样式)

用于设置单元格内容字体格式的注解

| 参数                     | 含义               |
| ------------------------ | ------------------ |
| **`fontName`**           | 字体名称           |
| **`fontHeightInPoints`** | 字体高度           |
| **`italic`**             | 是否斜体           |
| **`strikeout`**          | 是否设置删除水平线 |
| **`color`**              | 字体颜色           |
| **`typeOffset`**         | 偏移量             |
| **`underline`**          | 下划线             |
| **`bold`**               | 是否加粗           |
| **`charset`**            | 编码格式           |

### @ContentLoopMerge(合并单元格)

设置合并单元格的注解

| 参数         | 含义 |
| ------------ | ---- |
| eachRow      |      |
| columnExtend |      |

### @ContentRowHeight(设置行高)

| 参数  | 含义                   |
| ----- | ---------------------- |
| value | 行高，`-1`代表自动行高 |

### @ContentStyle(设置内容格式)

| 参数                      | 含义                                                         |
| ------------------------- | ------------------------------------------------------------ |
| **`dataFormat`**          | 日期格式                                                     |
| **`hidden`**              | 设置单元格使用此样式隐藏                                     |
| **`locked`**              | 设置单元格使用此样式锁定                                     |
| **`quotePrefix`**         | 在单元格前面增加`符号，数字或公式将以字符串形式展示          |
| **`horizontalAlignment`** | 设置是否水平居中                                             |
| **`wrapped`**             | 设置文本是否应换行。将此标志设置为`true`通过在多行上显示使单元格中的所有内容可见 |
| **`verticalAlignment`**   | 设置是否垂直居中                                             |
| **`rotation`**            | 设置单元格中文本旋转角度。03版本的Excel旋转角度区间为-90° ~ 90°，07版本的Excel旋转角度区间为0°~180° |
| **`indent`**              | 设置单元格中缩进文本的空格数                                 |
| **`borderLeft`**          | 设置左边框的样式                                             |
| **`borderRight`**         | 设置右边框样式                                               |
| **`borderTop`**           | 设置上边框样式                                               |
| **`borderBottom`**        | 设置下边框样式                                               |
| **`leftBorderColor`**     | 设置左边框颜色                                               |
| **`rightBorderColor`**    | 设置右边框颜色                                               |
| **`topBorderColor`**      | 设置上边框颜色                                               |
| **`bottomBorderColor`**   | 设置下边框颜色                                               |
| **`fillPatternType`**     | 设置填充类型                                                 |
| **`fillBackgroundColor`** | 设置背景色                                                   |
| **`fillForegroundColor`** | 设置前景色                                                   |
| **`shrinkToFit`**         | 设置自动单元格自动大小                                       |

### @HeadFontStyle(定制标题字体格式)

| 参数                     | 含义             |
| ------------------------ | ---------------- |
| **`fontName`**           | 设置字体名称     |
| **`fontHeightInPoints`** | 设置字体高度     |
| **`italic`**             | 设置字体是否斜体 |
| **`strikeout`**          | 是否设置删除线   |
| **`color`**              | 设置字体颜色     |
| **`typeOffset`**         | 设置偏移量       |
| **`underline`**          | 设置下划线       |
| **`charset`**            | 设置字体编码     |
| **`bold`**               | 设置字体是否家畜 |

### @HeadRowHeight(设置标题行行高)

| 参数        | 含义                       |
| ----------- | -------------------------- |
| **`value`** | 设置行高，`-1`代表自动行高 |

### @HeadStyle(设置标题样式)

| 参数                      | 含义                                                         |
| ------------------------- | ------------------------------------------------------------ |
| **`dataFormat`**          | 日期格式                                                     |
| **`hidden`**              | 设置单元格使用此样式隐藏                                     |
| **`locked`**              | 设置单元格使用此样式锁定                                     |
| **`quotePrefix`**         | 在单元格前面增加`符号，数字或公式将以字符串形式展示          |
| **`horizontalAlignment`** | 设置是否水平居中                                             |
| **`wrapped`**             | 设置文本是否应换行。将此标志设置为`true`通过在多行上显示使单元格中的所有内容可见 |
| **`verticalAlignment`**   | 设置是否垂直居中                                             |
| **`rotation`**            | 设置单元格中文本旋转角度。03版本的Excel旋转角度区间为-90° ~ 90°，07版本的Excel旋转角度区间为0°~180° |
| **`indent`**              | 设置单元格中缩进文本的空格数                                 |
| **`borderLeft`**          | 设置左边框的样式                                             |
| **`borderRight`**         | 设置右边框样式                                               |
| **`borderTop`**           | 设置上边框样式                                               |
| **`borderBottom`**        | 设置下边框样式                                               |
| **`leftBorderColor`**     | 设置左边框颜色                                               |
| **`rightBorderColor`**    | 设置右边框颜色                                               |
| **`topBorderColor`**      | 设置上边框颜色                                               |
| **`bottomBorderColor`**   | 设置下边框颜色                                               |
| **`fillPatternType`**     | 设置填充类型                                                 |
| **`fillBackgroundColor`** | 设置背景色                                                   |
| **`fillForegroundColor`** | 设置前景色                                                   |
| **`shrinkToFit`**         | 设置自动单元格自动大小                                       |

```java
    @HeadStyle(fillForegroundColor = 40)
    @ExcelProperty({"基础属性", "制造商(可输入数字0-9字母a-zA-Z,下划线_-以及汉字,最大长字符64位)(必填)"})
    private String manufacturer;
```

这里的`fillForegroundColor = 40`指的是`SKY_BLUE`

![在这里插入图片描述](https://img-blog.csdnimg.cn/212a2603f858454ea4bcfc997caa6cfe.png#pic_center)


```java
package org.apache.poi.ss.usermodel;

public enum IndexedColors {
    BLACK1(0),
    WHITE1(1),
    RED1(2),
    BRIGHT_GREEN1(3),
    BLUE1(4),
    YELLOW1(5),
    PINK1(6),
    TURQUOISE1(7),
    BLACK(8),
    WHITE(9),
    RED(10),
    BRIGHT_GREEN(11),
    BLUE(12),
    YELLOW(13),
    PINK(14),
    TURQUOISE(15),
    DARK_RED(16),
    GREEN(17),
    DARK_BLUE(18),
    DARK_YELLOW(19),
    VIOLET(20),
    TEAL(21),
    GREY_25_PERCENT(22),
    GREY_50_PERCENT(23),
    CORNFLOWER_BLUE(24),
    MAROON(25),
    LEMON_CHIFFON(26),
    LIGHT_TURQUOISE1(27),
    ORCHID(28),
    CORAL(29),
    ROYAL_BLUE(30),
    LIGHT_CORNFLOWER_BLUE(31),
    SKY_BLUE(40),
    LIGHT_TURQUOISE(41),
    LIGHT_GREEN(42),
    LIGHT_YELLOW(43),
    PALE_BLUE(44),
    ROSE(45),
    LAVENDER(46),
    TAN(47),
    LIGHT_BLUE(48),
    AQUA(49),
    LIME(50),
    GOLD(51),
    LIGHT_ORANGE(52),
    ORANGE(53),
    BLUE_GREY(54),
    GREY_40_PERCENT(55),
    DARK_TEAL(56),
    SEA_GREEN(57),
    DARK_GREEN(58),
    OLIVE_GREEN(59),
    BROWN(60),
    PLUM(61),
    INDIGO(62),
    GREY_80_PERCENT(63),
    AUTOMATIC(64);
}
```

### @ExcelIgnore

不将该字段转换成Excel

### @ExcelIgnoreUnannotated

没有注解的字段都不转换