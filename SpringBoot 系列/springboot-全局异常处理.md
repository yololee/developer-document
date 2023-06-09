# springboot-全局异常处理

## 一、异常分类

这里的异常分类从系统处理异常的角度看，主要分为俩类：业务异常和系统异常

### 1、业务异常

业务异常主要是一些可预见性异常，处理业务异常，用来提示用户的操作，提高系统的可操作性

常见的业务异常提醒：

- 请输入XXX
- XXX不能为空
- XXX重复，请更换

### 2、系统异常

系统异常主要是一些不可预见性异常，处理系统异常，可以让展示出一个友好的用户界面，不易给用户造成反感。如果是一个金融类系统，在用户界面出现一个系统异常的崩溃界面，很有可能直接导致用户流失。

常见的系统异常提示：

- 页面丢失404

- 服务器异常500

## 二、SpringBoot2.0中异常处理 

SpringBoot的项目已经对有一定的异常处理了，但是对于我们开发者而言可能就不太合适了，因此我们需要对这些异常进行统一的捕获并处理。SpringBoot中有一个`ControllerAdvice`的注解，使用该注解表示开启了全局异常的捕获，我们只需在自定义一个方法使用`ExceptionHandler`注解然后定义捕获异常的类型即可对这些捕获的异常进行统一的处理。

示例代码：

```java
@ControllerAdvice
public class MyExceptionHandler {

    @ExceptionHandler(value =Exception.class)
	public String exceptionHandler(Exception e){
		System.out.println("未知异常！原因是:"+e);
       	return e.getMessage();
    }
}

```

上述的示例中，我们对捕获的异常进行简单的二次处理，返回异常的信息，虽然这种能够让我们知道异常的原因，但是在很多的情况下来说，可能还是不够人性化，不符合我们的要求。那么我们这里可以通过自定义的异常类以及枚举类来实现我们想要的那种数据吧

### pom.xml

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
```

### 自定义状态码

```java
package com.yolo.exception.handler.common;

public enum ApiStatus {
    /**
     * 操作成功
     */
    OK(200, "操作成功"),

    PARAM_ERROR(400, "参数错误"),

    /**
     * 未知异常
     */
    UNKNOWN_ERROR(500, "服务器出错啦");

    /**
     * 状态码
     */
    private final Integer code;
    /**
     * 内容
     */
    private final String message;

    ApiStatus(Integer code, String message) {
        this.code = code;
        this.message = message;
    }

    public Integer getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }
}
```

### 自定义异常类

```java
/**
 * 异常基类
 */
@Data
@EqualsAndHashCode(callSuper = true)
public class BaseException extends RuntimeException {
    private Integer code;
    private String message;

    public BaseException(ApiStatus status) {
        super(status.getMessage());
        this.code = status.getCode();
        this.message = status.getMessage();
    }

    public BaseException(Integer code, String message) {
        super(message);
        this.code = code;
        this.message = message;
    }
}
```

```java
/**
 * JSON异常
 */
@Getter
public class JsonException extends BaseException {

    public JsonException(ApiStatus status) {
        super(status);
    }

    public JsonException(Integer code, String message) {
        super(code, message);
    }
}
```

```java
/**
 * 空指针异常
 */
public class NullPointerException extends BaseException{

    public NullPointerException(ApiStatus status) {
        super(status);
    }

    public NullPointerException(Integer code, String message) {
        super(code, message);
    }
}
```

### 自定义数据格式

```java
package com.yolo.exception.handler.common;


import lombok.Data;

/**
 * 通用的 API 接口封装
 */
@Data
public class ApiResponse {
    /**
     * 状态码
     */
    private Integer code;

    /**
     * 返回内容
     */
    private String message;

    /**
     * 返回数据
     */
    private Object data;

    /**
     * 无参构造函数
     */
    private ApiResponse() {

    }

    /**
     * 全参构造函数
     *
     * @param code    状态码
     * @param message 返回内容
     * @param data    返回数据
     */
    private ApiResponse(Integer code, String message, Object data) {
        this.code = code;
        this.message = message;
        this.data = data;
    }


    /**
     * 构造一个自定义的API返回
     *
     * @param code    状态码
     * @param message 返回内容
     * @param data    返回数据
     * @return ApiResponse
     */
    public static ApiResponse of(Integer code, String message, Object data) {
        return new ApiResponse(code, message, data);
    }

    /**
     * 构造一个成功且带数据的API返回
     *
     * @param data 返回数据
     * @return ApiResponse
     */
    public static ApiResponse ofSuccess(Object data) {
        return of(ApiStatus.OK.getCode(), ApiStatus.OK.getMessage(), data);
    }

    /**
     * 构造一个成功且不带数据的API返回
     * @return ApiResponse
     */
    public static ApiResponse ofSuccess() {
        return of(ApiStatus.OK.getCode(), ApiStatus.OK.getMessage(), null);
    }

    /**
     * 构造一个异常且带数据的API返回
     * @return ApiResponse
     */
    public static  ApiResponse ofException(ApiStatus apiStatus, Object data) {
        return of(apiStatus.getCode(), apiStatus.getMessage(), data);
    }

    /**
     * 构造一个异常且不带数据的API返回
     * @return ApiResponse
     */
    public static  ApiResponse ofException(ApiStatus apiStatus) {
        return ofException(apiStatus, null);
    }
}
```

### 自定义全局异常处理类

```java
package com.yolo.exception.handler.handler;

import com.yolo.exception.handler.exception.JsonException;
import com.yolo.exception.handler.exception.NullPointerException;
import com.yolo.exception.handler.model.ApiResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;

/**
 * 统一异常处理
 */
@ControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    private static final String DEFAULT_ERROR_VIEW = "error";

    /**
     * 统一 json 异常处理
     *
     * @param exception JsonException
     * @return 统一返回 json 格式
     */
    @ExceptionHandler(value = JsonException.class)
    @ResponseBody
    public ApiResponse jsonErrorHandler(JsonException exception) {
        log.error("【JsonException】:{}", exception.getMessage());
        return ApiResponse.ofException(exception);
    }

    /**
     * 处理空指针的异常
     * @param e 空制造异常
     * @return 统一返回 json 格式
     */
    @ExceptionHandler(value = NullPointerException.class)
    @ResponseBody
    public ApiResponse exceptionHandler(NullPointerException e){
        log.error("发生空指针异常！原因是:",e);
        return ApiResponse.ofException(e);
    }

}
```

### Controller 控制层

```java
package com.yolo.exception.handler.handler;

import com.yolo.exception.handler.common.ApiResponse;
import com.yolo.exception.handler.common.ApiStatus;
import com.yolo.exception.handler.exception.JsonException;
import com.yolo.exception.handler.exception.NullPointerException;
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.validation.BindException;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.MissingServletRequestParameterException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.ResponseStatus;

import javax.validation.UnexpectedTypeException;
import java.util.ArrayList;
import java.util.List;

/**
 * 统一异常处理
 */
@ControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    /**
     * 统一 json 异常处理
     *
     * @param exception JsonException
     * @return 统一返回 json 格式
     */
    @ExceptionHandler(value = JsonException.class)
    @ResponseBody
    public ApiResponse jsonErrorHandler(JsonException exception) {
        log.error("【JsonException】:{}", exception.getMessage());
        return ApiResponse.ofException(ApiStatus.UNKNOWN_ERROR,exception.getLocalizedMessage());
    }

    /**
     * 处理空指针的异常
     * @param exception 空制造异常
     * @return 统一返回 json 格式
     */
    @ExceptionHandler(value = NullPointerException.class)
    @ResponseBody
    public ApiResponse nullPointExceptionHandler(NullPointerException exception){
        log.error("发生空指针异常！原因是:",exception);
        return ApiResponse.ofException(ApiStatus.UNKNOWN_ERROR,exception.getLocalizedMessage());
    }

    /**
     * 全局异常
     */
    @ExceptionHandler
    @ResponseBody
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ApiResponse exceptionHandler(Exception e) {
        log.error("服务器出现未知错误", e);
        return ApiResponse.ofException(ApiStatus.UNKNOWN_ERROR);
    }


    /**
     * {@code @Valid}参数校验失败异常
     */
    @ExceptionHandler
    @ResponseBody
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiResponse exceptionHandler(MethodArgumentNotValidException e) {
        log.error("参数校验失败异常", e);
        return ApiResponse.ofException(ApiStatus.PARAM_ERROR,extractError(e.getBindingResult()));
    }

    /**
     * {@code @Valid}参数校验失败异常
     */
    @ExceptionHandler
    @ResponseBody
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiResponse exceptionHandler(BindException e) {
        log.error("参数校验失败异常", e);
        return ApiResponse.ofException(ApiStatus.PARAM_ERROR,extractError(e.getBindingResult()));
    }

    /**
     * 无效的参数异常
     */
    @ExceptionHandler
    @ResponseBody
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiResponse exceptionHandler(IllegalArgumentException e) {
        log.error("无效的参数异常", e);
        return ApiResponse.ofException(ApiStatus.PARAM_ERROR,e.getLocalizedMessage());
    }

    /**
     * {@code @Valid}参数校验失败异常
     */
    @ExceptionHandler
    @ResponseBody
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiResponse exceptionHandler(MissingServletRequestParameterException e) {
        log.error("参数校验失败异常", e);
        return ApiResponse.ofException(ApiStatus.PARAM_ERROR,e.getLocalizedMessage());
    }

    /**
     * 无效的参数异常
     */
    @ExceptionHandler
    @ResponseBody
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiResponse exceptionHandler(UnexpectedTypeException e) {
        log.error("参数校验失败异常", e);
        return ApiResponse.ofException(ApiStatus.PARAM_ERROR,e.getLocalizedMessage());
    }

    /**
     * 从绑定结果中提出错误字段
     */
    private List<FieldError> extractError(BindingResult bindingResult) {
        List<FieldError> fieldErrors = new ArrayList<>();
        bindingResult.getFieldErrors().forEach(fieldError -> {
            FieldError error = new FieldError();
            error.setField(fieldError.getField());
            error.setRejectedValue(fieldError.getRejectedValue());
            error.setDefaultMessage(fieldError.getDefaultMessage());
            fieldErrors.add(error);
        });
        return fieldErrors;
    }

    @Data
    private static class FieldError {

        private String field;
        private Object rejectedValue;
        private String defaultMessage;

    }

}
```

### 功能测试

![image-20230504155012069](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230504155012069.png)


> 注意：
>
> 自义定全局异常处理除了可以处理上述的数据格式之外，也可以处理页面的跳转，只需在新增的异常方法的返回处理上填写该跳转的路径并不使用`ResponseBody` 注解即可。 细心的同学也许发现了在`GlobalExceptionHandler`类中使用的是`ControllerAdvice`注解，而非`RestControllerAdvice`注解，如果是用的`RestControllerAdvice`注解，它会将数据自动转换成JSON格式，这种于`Controller`和`RestController`类似，所以我们在使用全局异常处理的之后可以进行灵活的选择处理。

>  [Gitee项目地址（demo-exception-handler）](https://gitee.com/huanglei1111/yolo-springboot-demo/tree/master/demo-exception-handler)

