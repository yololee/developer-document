springboot：自定义注解使用SPEL表达式动态获取方法传参

### 自定义注解如下

```java
package com.yolo.framework.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 *  jobDesc author executorHandler 不能以 # 开头 不然spel表达式无法动态获取值
 */
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface XxlRegister {

    String executorHandler();

    String cron();

    String jobDesc() default "default jobDesc";

    String author() default "default Author";

    /*
     * 默认为 ROUND 轮询方式
     * 可选： FIRST 第一个
     * */
    String executorRouteStrategy() default "ROUND";

    //调度状态：0-停止，1-运行
    int triggerStatus() default 0;
}
```

### 使用

```java
@Component
@Slf4j
public class TestXxlRegisterService {
    @XxlRegister(executorHandler = "demo-auto-register-test",cron = "#user.corn", author = "yolo", jobDesc = "测试auto-register")
    public void execute(User user) {
        log.info("执行了");
    }

}
```

这里的corn表达式我希望动态根据user对象里面的字段动态传值

**如果是单个参数,@XxlRegister(cron = "#corn")
如果是对象参数,@XxlRegister(cron = "#user.corn")**

### 注解切面实现

> 使用 **SpelExpressionParser 和DefaultParameterNameDiscoverer 来动态获取参数**

```java
package com.yolo.framework.aspectj;

import cn.hutool.core.collection.CollUtil;
import cn.hutool.core.map.MapUtil;
import cn.hutool.core.util.StrUtil;
import com.yolo.framework.annotation.XxlRegister;
import com.yolo.framework.extend.xxljob.model.XxlJobGroup;
import com.yolo.framework.extend.xxljob.model.XxlJobInfo;
import com.yolo.framework.extend.xxljob.service.JobGroupService;
import com.yolo.framework.extend.xxljob.service.JobInfoService;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.DefaultParameterNameDiscoverer;
import org.springframework.expression.EvaluationContext;
import org.springframework.expression.Expression;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import org.springframework.expression.spel.support.StandardEvaluationContext;
import org.springframework.stereotype.Component;
import java.util.*;

@Slf4j
@Aspect
@Component
public class XxlRegisterAspect{

    @Autowired
    private JobInfoService jobInfoService;

    @Autowired
    private JobGroupService jobGroupService;


    private final SpelExpressionParser parserSpEl = new SpelExpressionParser();
    private final DefaultParameterNameDiscoverer parameterNameDiscoverer= new DefaultParameterNameDiscoverer();

    @AfterReturning(pointcut = "@annotation(xxlRegister)")
    public void doAfter(JoinPoint joinPoint, XxlRegister xxlRegister){
        //注册任务
        addJobInfo(joinPoint,xxlRegister);
    }

    private void addJobInfo(JoinPoint joinPoint,XxlRegister xxlRegister) {

        HashMap<String, String> hashMap = MapUtil.newHashMap();
        hashMap.putIfAbsent("cron",generateKeyBySpEL(xxlRegister.cron(), joinPoint));
        hashMap.putIfAbsent("executorHandler",generateKeyBySpEL(xxlRegister.executorHandler(), joinPoint));
        hashMap.putIfAbsent("jobDesc",generateKeyBySpEL(xxlRegister.jobDesc(), joinPoint));
        hashMap.putIfAbsent("author",generateKeyBySpEL(xxlRegister.author(), joinPoint));
        hashMap.putIfAbsent("executorRouteStrategy",generateKeyBySpEL(xxlRegister.executorRouteStrategy(), joinPoint));
        hashMap.putIfAbsent("triggerStatus",generateKeyBySpEL(xxlRegister.triggerStatus() + "", joinPoint));
    }

    public String generateKeyBySpEL(String key, JoinPoint joinPoint) {
        if (StrUtil.startWith(key,"#")){
            Expression expression = parserSpEl .parseExpression(key);
            EvaluationContext context = new StandardEvaluationContext();
            MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
            Object[] args = joinPoint.getArgs();
            String[] paramNames = parameterNameDiscoverer.getParameterNames(methodSignature.getMethod());
            for(int i = 0 ; i < args.length ; i++) {
                if (paramNames != null){
                    context.setVariable(paramNames[i], args[i]);
                }
            }
            return Objects.requireNonNull(expression.getValue(context)).toString();
        }
        return key;
    }
}

```

