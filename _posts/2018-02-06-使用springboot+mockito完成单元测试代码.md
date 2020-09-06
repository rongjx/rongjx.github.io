---
layout:     post
title:      springboot+mockito单元测试
subtitle:   使用springboot+mockito完成单元测试代码
date:       2018-02-06
author:     wind
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Blog
---
# 使用背景
单元测试在写代码的过程中非常重要,不是必须但是有了单元测试,一次编写可以后面很方便的对所有已写好的代码进行测试,从而保障代码的质量,减少 bug ,单元测试的主要目的在于在数据库不产生脏数据的前提下能够快速验证代码逻辑的正确性

# 后端单元测试
## 场景一：发送http请求到控制层
这个场景其实一般使用 postman 等其他工具也能完成,也就是调用系统真实正式接口,一般可以不写,但是在代码中写了后可以方便后续测试,一次书写多次使用

```
package com.dotoyo.archivedb.ctl;

import com.dotoyo.archivedb.ArchivedbApplication;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.http.ResponseEntity;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;

import java.net.URL;
import java.util.HashMap;
import java.util.Map;

/**
 * 注意：此处只是方便发送post请求 会真实产生数据
 */
@RunWith(SpringRunner.class)
@Transactional
@SpringBootTest(classes = ArchivedbApplication.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class ControllerHttpTest {

    /**
     * @LocalServerPort 提供了 @Value("${local.server.port}") 的代替
     */
    @LocalServerPort
    private int port;
    private URL base;

    /**
     * 发送请求
     */
    @Autowired
    private TestRestTemplate restTemplate;

    @Before
    public void setUp() throws Exception {
        String url = String.format("http://localhost:%d/", port);
        System.out.println("单元测试启动开始!!!!!");
        System.out.println("baseUrl= " + url);
        this.base = new URL(url);
    }

    /**
     * 发送post请求
     *
     * @param urlPath
     * @param params
     * @return
     */
    ResponseEntity<String> postSend(String urlPath, MultiValueMap<String, String> params) {
        String url = this.base.toString() + urlPath;
        ResponseEntity<String> response = restTemplate.postForEntity(url, params, String.class, "");
        return response;
    }


    /**
     * 发送get请求
     *
     * @param urlPath
     * @param map
     * @return
     */
    ResponseEntity<String> getSend(String urlPath, HashMap<String, String> map) {
        String url = this.base.toString() + urlPath;
        if (map != null && map.size() > 0) {
            url += "?";
        }
        for (Map.Entry<String, String> entry : map.entrySet()) {
            url += entry.getKey() + "=" + entry.getValue() + "&";
        }
        ResponseEntity<String> response = restTemplate.getForEntity(url, String.class, "");
        return response;
    }


    /**
     * 测试用例1 post请求
     */
    @Test
    public void test1() {
        MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
        postSend("test", params);
        Assert.assertTrue("通过单元测试", true);
    }
}
```

## 场景二：控制层验证代码逻辑
这块主要考虑一些注入的 service 如何模拟，特别是一些外部 service,可使用 Mockito 来进行模拟

```
package com.dotoyo.archivedb.ctl;

import com.dotoyo.archivedb.entity.User;
import com.dotoyo.archivedb.service.IUserService;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mock;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.Date;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;

/**
 * service为mock 不会真实产生数据
 */
@RunWith(SpringRunner.class)
public class ControllerTest {

    /**
     * 依赖使用的mockService
     */
    @Mock
    private IUserService userService;

    @Before
    public void setUp() {
        System.out.println("单元测试启动开始!!!!!");
        initServiceDo();
        System.out.println("mockService方法行为设置完成");
    }

    /**
     * 定义mock Service 方法行为返回值等等
     */
    private void initServiceDo() {
        when(userService.insert(any(), any())).thenReturn(true);
    }


    /**
     * 测试用例1 控制层逻辑
     */
    @Test
    public void test1() {
        User users = new User();
        users.setBirthday(new Date());
        users.setCreateDate(new Date());
        userService.insert(users, User.class);
        Assert.assertTrue("通过单元测试", true);
    }
}
```

## 场景三：验证业务组合层逻辑
一般服务拆分后，都会有一个业务组装层,这一层主要也是要依赖一些 service,也是使用 Mockito 来进行模拟

```
package com.dotoyo.archivedb.busiService;

import com.dotoyo.archivedb.entity.User;
import com.dotoyo.archivedb.service.IUserService;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mock;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.Date;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;

/**
 * service为mock 不会真实产生数据
 * 业务组合层单元测试类
 */
@RunWith(SpringRunner.class)
public class BusiServiceTest {

    /**
     * 依赖使用的mockService
     */
    @Mock
    private IUserService userService;

    @Before
    public void setUp() {
        System.out.println("单元测试启动开始!!!!!");
        initServiceDo();
        System.out.println("mockService方法行为设置完成");
    }

    /**
     * 定义mock Service 方法行为返回值等等
     */
    private void initServiceDo() {
        when(userService.insert(any(), any())).thenReturn(true);
    }


    /**
     * 测试用例1 组合业务层逻辑
     */
    @Test
    public void test1() {
        User users = new User();
        users.setBirthday(new Date());
        users.setCreateDate(new Date());
        userService.insert(users, User.class);
        Assert.assertTrue("通过单元测试", true);
    }
}
```
## 场景四：基础DB层测试验证
这一层主要是验证 DB 层的配置啊,写入数据库的逻辑是否正确等,避免产生测试脏数据可以配置 JUnit 测试框架使用,开启事务后会默认回滚数据

```
package com.dotoyo.archivedb.dbServer;

import com.dotoyo.archivedb.ArchivedbApplication;
import com.dotoyo.archivedb.entity.User;
import com.dotoyo.archivedb.service.IUserService;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Transactional;

/**
 * db层service单元测试类 数据库不会产生真实数据
 */
@RunWith(SpringRunner.class)
@Transactional
@SpringBootTest(classes = ArchivedbApplication.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class DbServiceTest {

    @Autowired
    private IUserService userService;

    @Before
    public void setUp() throws Exception {
        System.out.println("单元测试启动开始!!!!!");
    }

    /**
     * 测试用例1
     */
    @Test
    public void test1() {
        userService.insert(new User(), User.class);
        Assert.assertTrue("通过单元测试", true);
    }
}
```

# 单元测试的必要性
书写单元测试代码等同于拷贝了同样一份逻辑实现代码，同时为了不依赖一些服务，还需要对一些 mock 服务进行设置，对里面一些方法配置行为，返回值等等，无疑会增加很多额外的代码工作量，所以对于单元测试代码很多人一开始是非常抗拒的，但是在实际应用后发现有了单元测试代码在实际个人代码质量还有问题定位上会非常方便

- 1.如果不写单元测试代码，经常会产生一些脏数据到数据库，后面清理会花费一些时间
- 2.有了单元测试代码我们后面自己对已完成的一些功能接口，服务的逻辑快速验证，减少因为书写后面的代码影响了前面已完成功能的影响
- 3.有了单元测试，自测的质量会高很多，当产品需求改动后，代码改动后，单元测试代码其实只需要拷贝一下逻辑改动的模块即可快速验证，减少代码出错率
- 4.基于上面这些优势，自测通过率高，验证方便我们书写的接口其实问题很少，从而减少查错和返工，最后会发现其实节省出来的时间远远大于书写单元测试代码额外花费的时间

所以日常开发还是尽可能的书写单元测试来配合完成代码的自测。



