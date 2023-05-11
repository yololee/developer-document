# idea-类注释和方法注释

## 一、类注释

1.File-->settings-->Editor-->File and Code Templates-->Files

![image-20230511143413236](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511143413236.png)

2.选择includes在空白处进行粘贴修改

```java
/**
   * @ClassName ${NAME} 
   * @Description TODO
   * @Author hl
   * @Date ${DATE} ${TIME}
   * @Version 1.0
   */ 
```

![image-20230511143433888](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511143433888.png)


3. 实际效果

![image-20230511143445615](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511143445615.png)


## 二、方法注释

### 1.创建模板

File-->Settings-->Editor-->Live Templates

![image-20230511143522055](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511143522055.png)


> methodTemplates 是我们的组名，然后在这个组下添加

![image-20230511143536859](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511143536859.png)


### 2.设置模板内容

```java
**
   * @MethodName $title$
   * @Description $description$ $param$ $return$ $throws$
   * @Author hl
   * @Date $date$ $TIME$
   */
```

> 代码的第一行必须要以* *开头，而不能以 / * *开头，否则会出现参数和返回值为空的情况

![image-20230511143548777](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511143548777.png)


### 3.设置模板应用场景

![image-20230511143602456](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511143602456.png)


> 如果曾经修改过，则显示为change而不是define

### 4.设置参数

第3步和第4步顺序不可颠倒，否则第5步将获取不到方法

![image-20230511143647217](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511143647217.png)


> param

```java
groovyScript("def result = '';def params = \"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList(); for(i = 0; i < params.size(); i++) {if(params[i] != '')result+='  * @param ' + params[i] + ((i < params.size() - 1) ? '\\r\\n ' : '')}; return result == '' ? null : '\\r\\n ' + result", methodParameters()) 
```

> return

```java
  groovyScript("def result=''; def data=\"${_1}\"; def stop=false; if(data==null || data=='null' || data=='' || data=='void' ) { stop=true; }; if(!stop) { result += '\\r\\n' + '   * @return: ' + data; }; return result;", methodReturnType())
```

### 三、去掉注释警告

file-settings-editor-inspections-java-javadoc

![image-20230511143710616](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230511143710616.png)


> 输入警告字段的名称