# springboot-整合validator之自定义注解

> pom.xml

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <dependency>
            <groupId>javax.validation</groupId>
            <artifactId>validation-api</artifactId>
            <version>2.0.1.Final</version>
        </dependency>

        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.4.5</version>
        </dependency>
```

## 自定义注解(`@TextFormat`)-字符串校验

> 自定义注解

```java
package com.yolo.validator.common.validator.annotation;

import com.yolo.validator.common.validator.handler.TextFormatHandler;

import javax.validation.Constraint;
import javax.validation.Payload;
import java.lang.annotation.*;

/**
 * 自定义参数校验注解： @TextFormat
 * @author jujueaoye
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = TextFormatHandler.class)
@Target({ElementType.PARAMETER, ElementType.FIELD, ElementType.TYPE})
public @interface TextFormat {

    /**
     * 是否必填 默认是必填的
     */
    boolean required() default true;

    /**
     * 判断字符串长度
     */
    int maxLength() default -1;


    /**
     * 判断字符串中是否包含中文 包含则抛出异常
     */
    boolean notChinese() default false;

    /**
     * 是否包含,不包含抛出异常
     */
    String[] contains() default {};

    /**
     * 是否不包含,包含抛出异常
     */
    String[] notContains() default {};

    /**
     * 前缀以xx开始
     */
    String startWith() default "";

    /**
     * 后缀以xx结束
     */
    String endsWith() default "";

    /**
     * 默认错误提示信息
     */
    String message() default "参数校验失败!";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}

```

> 实现方法

```java
package com.yolo.validator.common.validator.handler;


import cn.hutool.core.collection.CollUtil;
import cn.hutool.core.collection.ListUtil;
import cn.hutool.core.util.StrUtil;
import com.yolo.validator.common.dto.ApiStatus;
import com.yolo.validator.common.exception.ParamException;
import com.yolo.validator.common.validator.annotation.TextFormat;
import org.springframework.util.StringUtils;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;
import java.util.*;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
import java.util.stream.Collectors;

/**
 * 参数校验验证器
 */
public class TextFormatHandler implements ConstraintValidator<TextFormat, String> {

    private Boolean  required;
    private int maxLength;
    private boolean notChinese;
    private Set<String> contains;
    private Set<String> notContains;
    private String startWith;
    private String endsWith;

    // 注解初始化时执行
    @Override
    public void initialize(TextFormat textFormat) {
        this.required = textFormat.required();
        this.maxLength = textFormat.maxLength();
        this.notChinese = textFormat.notChinese();
        this.contains = Arrays.stream(textFormat.contains()).collect(Collectors.toSet());
        this.notContains = Arrays.stream(textFormat.notContains()).collect(Collectors.toSet());
        this.startWith = textFormat.startWith();
        this.endsWith = textFormat.endsWith();
    }

    @Override
    public boolean isValid(String target, ConstraintValidatorContext context) {
        if (StrUtil.isBlank(target)) {
            return !required;
        }

        if (!required && StrUtil.isNotBlank(target) && maxLength != -1 && target.length() < maxLength){
            return Boolean.TRUE;
        }

        if (StrUtil.isNotBlank(target) && notChinese){
            return checkNoeChinese(target);
        }

        if (CollUtil.isNotEmpty(contains) && contains.contains(target)){
           return Boolean.TRUE;
        }

        if (CollUtil.isNotEmpty(notContains) && !notContains.contains(target)){
            return Boolean.TRUE;
        }

        if (StrUtil.isNotBlank(startWith) && target.startsWith(startWith)) {
            return Boolean.TRUE;
        }

        if (StrUtil.isNotBlank(endsWith) && target.endsWith(startWith)) {
            return Boolean.TRUE;
        }

        return Boolean.FALSE;
    }

    private Boolean checkNoeChinese(String target) {
        String regEx = "[\\u4e00-\\u9fa5]";
        Pattern pattern = Pattern.compile(regEx);
        Matcher matcher = pattern.matcher(target);
        return  matcher.find();
    }

}
```

## 自定义注解(`@NumberCheck`)-数字校验

> 自定义注解

```java
package com.yolo.validator.common.validator.annotation;




import com.yolo.validator.common.validator.handler.NumberCheckValidator;

import javax.validation.Constraint;
import javax.validation.Payload;
import java.lang.annotation.*;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = NumberCheckValidator.class)
@Target({ElementType.PARAMETER, ElementType.FIELD, ElementType.TYPE})
public @interface NumberCheck {

    /**
     * 是否必填 默认是必填的
     */
    boolean required() default true;

    /**
     * 最小值
     */
    int min() default 0;

    /**
     * 最大值
     */
    int max() default 1000000;

    /**
     * 最大小数点位数
     */
    int maxScale() default 2;
    /**
     * 默认错误提示信息
     */
    String message() default "参数校验失败!";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}

```

> 实现方法

```java
package com.yolo.validator.common.validator.handler;

import cn.hutool.core.convert.Convert;
import cn.hutool.core.util.ObjectUtil;
import com.yolo.validator.common.validator.annotation.NumberCheck;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

public class NumberCheckValidator implements ConstraintValidator<NumberCheck, Object> {

    private Boolean  required;
    private int min;
    private int max;
    private int maxScale;

    @Override
    public void initialize(NumberCheck numberCheck) {
        this.required = numberCheck.required();
        this.min = numberCheck.min();
        this.max = numberCheck.max();
        this.maxScale = numberCheck.maxScale();
    }

    @Override
    public boolean isValid(Object target, ConstraintValidatorContext constraintValidatorContext) {
        if (ObjectUtil.isNull(target)  || Convert.toInt(target) <= 0) {
            return !required;
        }

        if (target instanceof Integer){
            int tar = (Integer) target;

            if (!required && (tar > min || tar < max)){
                return Boolean.TRUE;
            }
        }

        if (target instanceof Long){
            long tar = (Long) target;


            if (!required && (tar > min || tar < max)){
                return Boolean.TRUE;
            }
        }

        if (target instanceof Double){
            double tar = (Double) target;

            if (!required && (tar < min || tar > max)){
                return Boolean.FALSE;
            }

            int length = String.valueOf(tar).split("\\.")[1].length();
            if (!required &&  length <= maxScale){
                return Boolean.TRUE;
            }
        }

        return false;
    }
}

```

## 自定义注解(`@ListValue`)-集合校验

> 自定义注解

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.PARAMETER, ElementType.FIELD, ElementType.TYPE})
@Constraint(validatedBy = ListValueConstrainValidator.class)
public @interface ListValue {

    /**
     * 是否必填 默认是必填的
     */
    boolean required() default true;

    /**
     * 验证失败的消息
     */
    String message() default "集合校验失败";

    /**
     * 分组的内容
     */
    Class<?>[] groups() default {};
    /**
     * 错误验证的级别
     */
    Class<? extends Payload>[] payload() default {};


    int[] values() default {};
}
```

> 实现方法

```java
package com.yolo.validator.common.validator.handler;

import com.yolo.validator.common.validator.annotation.ListValue;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;
import java.util.Arrays;
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

public class ListValueConstrainValidator implements ConstraintValidator<ListValue,Integer> {
    
    private final Set<Integer> set = new HashSet<>();

    private Boolean  required;

    /**
     * 初始化
     *
     * @param constraintAnnotation 约束注释
     */
    @Override
    public void initialize(ListValue constraintAnnotation) {
        Arrays.stream(constraintAnnotation.values()).filter(Objects::nonNull).forEach(set::add);
        required = constraintAnnotation.required();
    }


    /**
     * 校验
     *
     * @param value           需要校验的值
     * @param constraintValidatorContext 约束验证器上下文
     * @return boolean
     */
    @Override
    public boolean isValid(Integer value, ConstraintValidatorContext constraintValidatorContext) {
        // 注解表明为必选项 则不允许为空，否则可以为空
        if (value == null) {
            return !required;
        }

        return set.contains(value);
    }
}
```

## 自定义注解(`@EnumCheck)-枚举校验

> 枚举类

```java
package com.yolo.validator.domain;

import java.util.Arrays;

/**
 * 需求类型枚举
 */
public enum DemandTypeEnum {


    /**
     * 测试
     */
    TEST(0, "测试"),

    /**
     * 正式
     */
    OFFICIAL(1, "正式");


    private final int key;
    private final String value;

    public int getKey() {
        return key;
    }

    public String getValue() {
        return value;
    }

    DemandTypeEnum(int key, String value) {
        this.key = key;
        this.value = value;
    }

    /**
     * 判断数值是否属于枚举类的值
     */
    public static boolean isInclude(Integer key) {
        return Arrays.stream(values()).anyMatch(e -> key == e.getKey());
    }

    public static String getName(int key) {
        String value = "";
        for (DemandTypeEnum e : values()) {
            if (e.key == key) {
                value = e.getValue();
                break;
            }
        }
        return value;
    }

}
```

> 自定义注解

```java
package com.yolo.validator.common.validator.annotation;

import com.yolo.validator.common.validator.handler.EnumCheckValidator;

import javax.validation.Constraint;
import javax.validation.Payload;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;


@Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = EnumCheckValidator.class)
public @interface EnumCheck {
    /**
     * 是否必填 默认是必填的
     */
    boolean required() default true;

    /**
     * 验证失败的消息
     */
    String message() default "枚举验证失败";

    /**
     * 分组的内容
     */
    Class<?>[] groups() default {};

    /**
     * 错误验证的级别
     */
    Class<? extends Payload>[] payload() default {};

    /**
     * 枚举的Class
     */
    Class<? extends Enum<?>> enumClass();

    /**
     * 枚举中的验证方法
     */
    String enumMethod() default "isInclude";

}
```

> 实现方法

```java
package com.yolo.validator.common.validator.handler;

import com.yolo.validator.common.validator.annotation.EnumCheck;
import lombok.extern.slf4j.Slf4j;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;
import java.lang.reflect.Method;


@Slf4j
public class EnumCheckValidator implements ConstraintValidator<EnumCheck, Object> {


    private EnumCheck enumCheck;

    @Override
    public void initialize(EnumCheck enumCheck) {
        this.enumCheck = enumCheck;
    }

    @Override
    public boolean isValid(Object value, ConstraintValidatorContext constraintValidatorContext) {
        // 注解表明为必选项 则不允许为空，否则可以为空
        if (value == null) {
            return !this.enumCheck.required();
        }

        Boolean result = Boolean.FALSE;
        Class<?> valueClass = value.getClass();
        try {
            //通过反射执行枚举类中validation方法
            Method method = this.enumCheck.enumClass().getMethod(this.enumCheck.enumMethod(), valueClass);
            result = (Boolean) method.invoke(null, value);
            if (result == null) {
                return false;
            }
        } catch (Exception e) {
            log.error("custom EnumCheckValidator error", e);
        }
        return result;
    }
}
```

