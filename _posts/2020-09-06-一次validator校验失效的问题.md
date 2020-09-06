---
layout:     post
title:      一次validator校验失效的问题
subtitle:    一次validator校验失效的问题
date:       2019-09-06
author:     wind
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Blog
---
# 背景
在springboot开发的项目控制层中有大量的参数校验，如何使用大量的if/else 等代码完成不够优雅而且很麻烦，这里我使用Java validator来进行参数优雅校验，Bean Validation是Java定义的一套基于注解的数据校验规范，目前已经从JSR 303的1.0版本升级到JSR 349的1.1版本，再到JSR 380的2.0版本（2.0完成于2017.08），已经经历了三个版本 。在`SpringBoot`中已经集成在 `starter-web`中，所以无需在添加其他依赖。

使用也很简单：

1.在控制层接口中导入 import org.springframework.validation.annotation.Validated

@Validated  注解标注为接口需要校验参数

```
 @PostMapping("/calc/submitTask")
    public APIResponse<Boolean> submitCalcTask(@Validated @RequestBody SubmitCalcTaskReqDTO reqDTO) {
        return APIResponse.returnOk();
    }
```

2.在对应的实体上对需要校验的参数和错误返回信息注解标注

```
@Data
@ApiModel
public class SubmitCalcTaskReqDTO {

    /**
     * 任务id
     */
    @NotNull(message = "taskId不能为空")
    Long taskId;

    /**
     * 是否模拟运行，模拟运行只打印日志
     */
    Boolean mock;

}
```



# 校验失效

上周因为安全扫描需要升级jar,对各个jar进行升级后，在测试接口的时候发现校验突然全部失效了，很奇怪，但是项目也不报错启动都很正常，最后在想估计是 jar 包升级引起的，后面一查果然是，我对spring-boot 从2.2.7.RELEASE 版本升级到  2.3.1.RELEASE ，一查官方才知道从2.3之后spring-boot-starter-web 去掉了hibernate-validator 这个jar包的依赖，导致校验不报错但是失效了,解决方案很简单直接引入校验 jar 包，或者加入官方匹配版本匹配的spring-boot-starter-validation

```
       <!-- 校验框架 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
```

spring-boot-starter-validation引入了 hibernate-validator 这个jar包，之后校验恢复正常。

