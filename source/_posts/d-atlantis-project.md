---
title: 【文档小记】Atlantis JMVT 游戏车队交流平台
tags:
  - Atlantis JMVT
  - Vue
  - SpringBoot
  - WebSite
  - 前端
  - 后端
toc: true
languages:
  - zh-CN
categories:
  - 文档小记
comments: false
cover: false
date: 2023-10-25 21:19:17
---

记录了这个 Vue + SpringBoot 前后端项目开发的一些知识点以及解决方案

<!-- more -->

> [GitHub 项目地址](https://github.com/CS0522/Atlantis)

## 基本介绍

### SpringBoot

* SpringBoot 是全新一代的 Spring 框架
* 开箱即用、预定优于配置、更轻量级
* 常用的第三方依赖整合（通过 Maven 子父工程的方式），简化 XML 配置，全部采用注解形式，内嵌 Web 应用容器
* AOP 编程的支持（AOP 面向切面编程），功能增强，注解形式
* IoC（Inversion of Control）控制反转
* 启动顺序
  * 调用构造函数进行数据初始化
  * 加载配置环境及上下文环境，返回应用配置上下文

### vue

* MVVM 设计模式
* V：视图 view
* ViewModel：一个同步 View 和 Model 的对象，连接 Model 和 View
* M：模型 model，在其中定义数据修改和操作的业务逻辑
* 双向数据绑定，通过 ViewModel 进行交互，Model 和 ViewModel 之间的交互是双向的，因此 View 数据的变化会同步到 Model 中，而 Model 数据的变化也会立即反应到 View 上

### MVC、MVVM 和三层架构

* MVC（设计模式）  
  MVC 通过分离 Model、View 和 Controller 的方式来组织代码结构。其中 View 负责页面的显示逻辑，Model 负责存储页面的业务数据，以及对相应数据的操作。并且 View 和 Model 应用了观察者模式，当 Model 层发生改变的时候它会通知有关 View 层更新页面。Controller 层是 View 层和 Model 层的纽带，它主要负责用户与应用的响应操作，当用户与页面产生交互的时候，Controller 中的事件触发器就开始工作了，通过调用 Model 层，来完成对 Model 的修改，然后 Model 层再去通知 View 层更新
* MVVM  
  Model 和 View 并无直接关联，而是通过 ViewModel 来进行联系的，Model 和 ViewModel 之间有着双向数据绑定的联系。因此当 Model 中的数据改变时会触发 View 层的刷新，View 中由于用户交互操作而改变的数据也会在 Model 中同步
* 三层架构（体系架构设计）
  * 高内聚，低耦合 
  * 表现层、业务逻辑层、数据访问层（SpringMVC、Spring、MyBatis）


### 项目介绍

Atlantis Racer JMVT 网站是由游戏爱好者组建的 Atlantis Racer JMVT 车队的网站。该网站项目采用 Vue + SpringBoot 前后端分离技术，从 0 开始开发，90% 的功能或组件自主实现，其他功能或组件采用了第三方方案，引用了 Element-UI, github-markdown-css, mavon-editor, vuex 等。由于第一次接触 Vue 和 SpringBoot 等技术，边学习、边练习、边实践，因此项目中代码有很多地方冗余、复杂或有更为简单的方案实现，还能够继续优化。网站部署地址：http://atlantisracer.tk（挂）。

开发环境：

Windows 11 + VSCode (前端、后端) + IDEA (后端) + Edge + Apifox (接口测试) + MySQL Workbench (数据库) + Git

前后端请求路径：

前端访问：`localhost:8080`

后端请求：

* localhost:8081/api
* localhost:8082/api（于 `utils/baseUrl.js`，`controller/*/application.yml` 下修改）

文件保存路径和数据库：

保存路径：

项目根目录 `/Data/{photos|article_pictures}/`（于 `com/atlantis/common/ProjectPath.java` 下修改）

测试用数据库：

`TestSamples_DB.sql`   

实际使用数据库：

`ForActualUse_DB.sql`

实现的功能包括：


### 数据库表关系

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/d-atlantis-project/admin.png)

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/d-atlantis-project/user.png)

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/d-atlantis-project/member.png)

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/d-atlantis-project/apply-user_admin.png)

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/d-atlantis-project/article-category.png)


### 路由关系

![](https://cdn.jsdelivr.net/gh/CS0522/CSBlog/source/_posts/d-atlantis-project/route.png)


### 总结

这次的项目开发，我们学习到了很多如：前端 Web 语言如 HTML、JavaScript、CSS 等；后端的语言 Java；前端的主流框架 Vue 和后端的主流框架 SpringBoot 相结合进行前后端的分离开发等。这些都是在理论课的基础上，进行知识的拓展和应用，让我们更进一步地了解企业中开发一个项目的大体流程，是宝贵的经验。


## 项目遇到的问题及解决方案  

### 组件固定于浏览器某一位置：  

利用 `position: fixed` 属性固定。 

```css
.top-bar
{
    /* 设置边框为圆角 */
    border-radius: 15px;
    float: left;
    text-align: left;
    /* overflow: hidden; */
    height: 150px;
    width: auto;
    /* top bar始终处于浏览器顶端 */
    position: fixed;
    /* 设置top bar位置 */
    top: 0cm;
    left: 1cm;
    right: 1cm;
    /* border: solid; */
    /* 设置top bar背景色 */
    background-color: rgba(252, 242, 241, 0.85);
    /* 设置top bar阴影 */
    box-shadow: 5px 15px 15px 5px #888888;
    /* 设置层级为1000，最大 */
    z-index: 1000;
}
```

### 禁止页面文字被复制：

```css
#box
{
  /* 设置全局文字不能被选中 */
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}
```

### 设置css作用范围：

```html
<style scoped src="@/../public/css/personalpage-style.css">
</style>
```

### <span style="color: red">（未解决）</span>获取当前路由路径并映射为中文：

  

### 路由跳转后，页面不刷新：  

给 `router-view` 设置 `:key` 属性，路由改变，就会重新渲染。  

```html
<template>
<div id="app">
    <keep-alive>
    <router-view v-if="$route.meta.keepAlive" :key="$route.fullPath"></router-view>
    </keep-alive>
    <router-view v-if="!$route.meta.keepAlive"></router-view>
</div>
</template>
```

### 子组件（子路由）向父组件（父路由）传递参数：

通过事件进行绑定，发生某个自定义事件，绑定一个方法，参数通过方法传递。

```html
<div id="dash-content-box">
<!-- 注意这个地方一定要绑定key！！！以后也是！！！ -->
<!-- 子组件传递参数，@绑定方法函数 -->
<router-view v-if="isRouterAlive"
                :key="$route.fullPath"
                @reload="reload"
                @show-dialog="showDialog">
</router-view>
</div>
```

```javascript
doOperation(operation, contentType, objId) {
    // 修改vuex中的state数据
    this.$store.commit('setOperation', operation);
    this.$store.commit('setContentType', contentType);
    this.$store.commit('setObjId', objId);
    // 向父路由emit一个show-dialog事件
    this.$emit('show-dialog', true)
}
```

### 兄弟组件（兄弟路由）数据通信：

使用 `vuex` 即可。需要注意 `vuex` 在页面刷新后数据丢失，因此不要存储登录相关数据。

```javascript
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.Store({
// vuex里的store刷新页面后内容会丢失
state: {
    // 用于DashContent子路由和AccountDialog之间的数据通信
    operation: '',
    objId: '',
    contentType: '',
    // 保存CRUD操作类型，以及目标item的id
    // 登录成功后保存用户信息(X! vuex里的刷新后内容丢失)
    // accountInfo: {},
    // isLogin: false,
    // loginType: '',
    // admins, users
},
mutations: {
    setOperation(state, newOperation){
        state.operation = newOperation;
    },
    setObjId(state, newId)
    {
        state.objId = newId;
    },
    setContentType(state, newContentType)
    {
        state.contentType = newContentType;
    },
},
})

export default store
```

### 手动刷新子组件（子路由）：

给子组件（子路由）设置 `v-if` 属性。自刷新可利用 `provide, inject` 注入函数依赖。刷新函数利用 `this.$nextTick(function() {})` 改变绑定 `v-if` 的boolean变量的值。

```html
<!-- 父组件内 -->
<TopBar v-if="!topBarReload"
        @reload="doTopBarReload()"></TopBar>
<router-view v-if="isRouterAlive"
            :key="$route.fullPath"
            @reload="reload"
            @show-dialog="showDialog"></router-view>
```

```javascript
data() {
    return {
        isShow: false,
        isRouterAlive: true,
        topBarReload: false,
    }
},
// 刷新路由操作
provide() {
    return {
        reload: this.reload,
        doTopBarReload: this.doTopBarReload,
    }     
},
// 刷新路由操作
reload() {
    this.isRouterAlive = false;
    this.$nextTick(function() {
    this.isRouterAlive = true;
    })
},
// 刷新topbar
doTopBarReload() {
    this.topBarReload = true;
    this.$nextTick(function() {
    this.topBarReload = false;
    })
}

// 子组件内
// 刷新路由操作，注入reload依赖
inject: ['reload'],
```

### cors跨域：

后端设置Config类，或者设置过滤器类。

```java
// Config class
@Configuration
public class CorsConfig {

    // 当前跨域请求最大有效时长。1天
    private static final long MAX_AGE = 24 * 60 * 60;

    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*"); // 1 设置访问源地址
        // corsConfiguration addAllowedOriginsPattern("*");
        corsConfiguration.addAllowedHeader("*"); // 2 设置访问源请求头
        corsConfiguration.addAllowedMethod("*"); // 3 设置访问源请求方法
        // corsConfiguration.setAllowCredentials(true); // 解决前后的session对象不一致问题
        corsConfiguration.setMaxAge(MAX_AGE);
        source.registerCorsConfiguration("/**", corsConfiguration); // 4 对接口配置跨域设置
        System.out.println("doing CORS config...");
        return new CorsFilter(source);
    }
}

// filter class. Need to add 'ServletComponentScan' at Application class
@WebFilter(filterName = "requestFilter", urlPatterns = {"/*"})
public class RequestFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain  filterChain)
        throws IOException, ServletException {

        System.out.println("doing request filter...");

        HttpServletResponse response = (HttpServletResponse) servletResponse;
        HttpServletRequest request = (HttpServletRequest) servletRequest;

        // 此处 setHeader、addHeader 方法都可用。但 addHeader时写多个会报错：“...,but only one is allowed”
        response.setHeader("Access-Control-Allow-Origin", "*");
        // response.addHeader("Access-Control-Allow-Origin", request.getHeader("origin"));
        // 解决预请求（发送2次请求），此问题也可在 nginx 中作相似设置解决。
        response.setHeader("Access-Control-Allow-Headers",
                "x-requested-with,Cache-Control,Pragma,Content-Type,Token, Content-Type");
        response.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, PUT, DELETE");
        response.setHeader("Access-Control-Max-Age", "3600");
        response.setHeader("Access-Control-Allow-Credentials", "true");
        String method = request.getMethod();
        if (method.equalsIgnoreCase("OPTIONS")) {
            servletResponse.getOutputStream().write("Success".getBytes("utf-8"));
        } else {
            filterChain.doFilter(servletRequest, servletResponse);
        }
    }

    @Override
    public void destroy() {
    }
}
```

### cors跨域后session id不一致：

`http://localhost:8080` 或 `http://172.16.12.103:8080` 只能成功一个。在 `application.yml` 中配置 Tomcat 监听ip地址。

```yml
server:
port: 8081
# 内网ip, 部署后填入外网ip
address: 172.16.12.103
```

```java
@Configuration
public class CorsConfig {

    // 当前跨域请求最大有效时长。1天
    private static final long MAX_AGE = 24 * 60 * 60;

    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration corsConfiguration = new CorsConfiguration();

        System.out.println("doing CORS config...");
        // corsConfiguration.addAllowedOrigin("*"); // 设置访问源地址
        //
        corsConfiguration.addAllowedOriginPattern("**");
        corsConfiguration.addAllowedHeader("*"); // 设置访问源请求头
        corsConfiguration.addAllowedMethod("*"); // 设置访问源请求方法
        //
        corsConfiguration.setAllowCredentials(true); // 解决前后的session对象不一致问题

        corsConfiguration.setMaxAge(MAX_AGE);
        source.registerCorsConfiguration("/**", corsConfiguration); // 对接口配置跨域设置
        return new CorsFilter(source);
    }
}
```

```javascript
const request = axios.create({
    baseURL: 'http://172.16.12.103:8081',
    timeout: 5000,
    // 解决前后session不一致问题
    withCredentials: true,
})
```

### 模块开发：

利用maven的继承和聚合。建立父工程，只包含 `pom.xml`，修改打包方式为 `pom`、声明子工程、统一设置依赖版本号以及子工程版本号。子工程内将 `<parent>` 设为父工程，`<dependency>` 中的依赖具有传递性。

```xml
<!-- 父工程中 -->
<!-- 修改打包方式 -->
<packaging>pom</packaging>
<!-- 统一设置版本号 -->
<properties>
    <java.version>11</java.version>
    <atlantis.version>0.0.1-SNAPSHOT</atlantis.version>
    <mybatis.version>2.2.2</mybatis.version>
    <mysql.version>8.0.30</mysql.version>
    <jackson.version>2.13.4</jackson.version>
    <druid.version>1.2.13-SNSAPSHOT</druid.version>
</properties>
<!-- dependency中进行引用 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>${druid.version}</version>
</dependency>
<!-- 声明子工程 -->
<modules>
    <module>controller</module>
    <module>mapper</module>
    <module>pojo</module>
    <module>service</module>
    <module>util</module>
    <module>exception</module>
    <module>common</module>
    <module>config</module>
    <module>filter</module>
</modules>
<!-- 子工程版本号 -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.atlantis</groupId>
            <artifactId>mapper</artifactId>
            <version>${atlantis.version}</version>
        </dependency>
        <dependency>
            <groupId>com.atlantis</groupId>
            <artifactId>pojo</artifactId>
            <version>${atlantis.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 代码重复，进行复用：

这个项目里，Java使用继承和泛型来减少代码重复；vue内主要重复的是axios请求，采用字符串拼接减少重复。
  
```java
// 关系：泛型ServiceImpl类实现泛型接口Service类。接口AdminService类继承泛型泛型接口Service类。AdminServiceImpl类继承泛型ServiceImp类，同时实现接口AdminService类。
// BaseService<T>
public interface BaseService<T> {
    public T getById(Integer id);
    public List<T> getAll();
    public boolean insert(T obj);
    public boolean update(T obj);
    public boolean updateInfo(T obj);
    public boolean updatePwd(T obj);
    public boolean delete(Integer id);
    public T login(T login);
}
// BaseServiceImpl<T>
public abstract class BaseServiceImpl<T> implements BaseService<T> {
    @Autowired
    // 代理对象自动装配，SpringBoot无法识别
    // userDao实例可能会报错，修改检查配置
    // protected 子类可继承
    protected BaseMapper<T> baseMapper;

    @Override
    public T getById(Integer id) {
        return baseMapper.getById(id);
    }

    @Override
    public List<T> getAll() {
        return new ArrayList<T>(baseMapper.getAll());
    }

    @Override
    public boolean insert(T obj) {
        return false;
    }

    @Override
    public boolean update(T obj) {
        return (baseMapper.update(obj) == 1);
    }

    @Override
    public boolean updateInfo(T obj) {
        return false;
    }

    @Override
    public boolean updatePwd(T obj)
    {
        return false;
    }

    @Override
    public boolean delete(Integer id) {
        return (baseMapper.delete(id) == 1);
    }

    @Override
    public T login(T login) {
        return null;
    }
}
// AdminService
public interface AdminService extends BaseService<Admin> {
}
// AdminServiceImpl
// IoC容器管理对象
@Service
public class AdminServiceImpl extends BaseServiceImpl<Admin> implements AdminService {

    @Override
    public boolean updateInfo(Admin admin) {
        return (baseMapper.updateInfo(admin) == 1);
    @Override
    public boolean updatePwd(Admin admin) {
        return (baseMapper.updatePwd(admin) == 1);
    }

    @Override
    public boolean insert(Admin admin) {
        // 查询数据库发现没有同名用户
        if (baseMapper.getByUsername(admin.getUsername()) == null)
        {
            return (baseMapper.insert(admin) == 1);
        }
        else
        {
            return false;
        }
    }

    @Override
    public Admin login(Admin login)
    {
        return (baseMapper.getByUsernameAndPassword(login.getUsername(), login.getPassword()));
    }
}
```

```javascript
load() 
{
    // 拼接字符串，减少代码重复
    let typeStr = '';
    if (this.type === 'admin') {
        typeStr = 'admins';
    }
    else if (this.type === 'user') {
        typeStr = 'users';
    }
    if (typeStr === '')
    {
        return;
    }
    // getAll请求，获取所有
    request.get("/" + typeStr).then(res => {
        if (res.code === code.GET_OK) {
            this.items = res.data;
        }
    }).catch(err => {
        console.log(err)
        this.$notify.error({
            title: '获取失败',
        });
    })
},
```

### 多个页面共用一个组件：

通过路径的变化，`props` 进行传参。`props` 内的参数JavaScript内无法直接获取，可以通过 `watch` 监听获得。

```javascript
// personal page router
{
    path: "/personal",
    redirect: "/personal/profile",
    component: Personal,
    // meta: {
    //     keepAlive: true // 需要缓存
    // },
    // props: true,
    children: [
        {
            path: ":type",
            component: PersonalContent,
            props: true
        }
    ]
}

// 组件内
props: ['type']
```
  
### `setTimeOut()` 延迟执行：

```javascript
// 延迟 300ms 刷新子路由
  setTimeout(() => {
      this.reload(); // 执行方法
  }, 300);
```

### 用户头像上传下载：
  
利用 `ElementUI` 上传控件，并修改样式。

```html
<!-- 界面 -->
<el-upload 
    class = "avatar-uploader" 
    :action = "actionUrl"
    :show-file-list = "false"
    :on-success = "handleAvatarSuccess" 
    :before-upload = "beforeAvatarUpload">
    <img v-if = "imageUrl" :src = "imageUrl" class = "avatar">
    <i v-else class = "el-icon-plus avatar-uploader-icon"></i>
</el-upload>
```

```javascript
// 用户头像上传
handleAvatarSuccess(res, file) {
    this.imageUrl = "http://localhost:8081/users/download/" + this.userId;
    this.reload();
    this.$notify.success({
        title: '头像更新成功'
    })
},
beforeAvatarUpload(file) {
    if (file) {
        console.log("uploading...actionUrl: " + this.actionUrl);
        const postfix = file.name.split('.')[1]
        const isSizeOk = file.size < (2 * 1024 * 1024);
        if (['png', 'jpeg', 'jpg'].indexOf(postfix) < 0) {
            this.$notify.error({
                title: '头像仅支持 .png, .jpg, .jpeg 格式'
            })
            this.$refs.upload.clearFiles()
            return false
        }
        if (!isSizeOk) {
            this.$notify.error({
                title: '上传头像大小不能超过 2MB'
            })
            return false
        }
        return file
    }
}
```

```java
// basePath定义在 application.yml 配置文件中
// 用户的头像上传
@PostMapping("/upload/{id}")
public Result upload(MultipartFile file, @PathVariable Integer id)
{
    System.out.println("uploading...");
    // 获取的file是临时文件，需要转存

    File filePath = new File(basePath);
    // 若目录不存在，创建
    if (!filePath.exists())
    {
        filePath.mkdirs();
    }

    // 获取原始文件名后缀
    // String postfix = Objects.requireNonNull(
    //                    file.getOriginalFilename()).substring(
    //                            (file.getOriginalFilename().lastIndexOf(".")));

    // 根据id重新生成用户名
    // 文件上传加入后缀 '.png'
    String fileName = "userId_" + id.toString() + ".png";

    // 转存文件
    try {
        file.transferTo(new File(basePath + fileName));
    }
    catch (IOException e) {
        throw new SystemException("upload failed", e, Code.SYS_ERR);
    }

    return new Result(Code.UPLOAD_OK, (String)fileName, "upload succeeded");
}

// 用户头像下载
@GetMapping("/download/{id}")
public Result download(HttpServletResponse response, @PathVariable Integer id)
{
    try
    {
        // 输入流读取文件内容
        FileInputStream fis = new FileInputStream(basePath + "userId_" + id.toString() + ".png");
        // 输出流写回浏览器
        ServletOutputStream sos = response.getOutputStream();
        response.setContentType("image/png");

        int len = 0;
        byte[] bytes = new byte[1024];
        while ((len = fis.read(bytes)) != -1)
        {
            sos.write(bytes, 0, len);
            sos.flush();
        }

        //close
        fis.close();
        sos.close();
    }
    catch (Exception e)
    {
        e.printStackTrace();
        return new Result(Code.DOWNLOAD_ERR, null, "no user photo");
    }

    return new Result(Code.DOWNLOAD_OK,
                    basePath + "userId_" + id.toString() + ".png",
                    "download succeeded");
}
```

### 分页查询：

利用 `ElementUI` 简化前端分页组件的开发；后端利用 `page-helper` 进行分页查询的业务层和控制层处理。

前端：

```html
<!-- pagination -->
<el-pagination style="zoom: 220%; margin-bottom: 20px"
            @size-change="handleSizeChange" 
            @current-change="handleCurrentChange" 
            :current-page.sync="currentPage"
            :page-size="pageSize" 
            layout="prev, pager, next, jumper" 
            :total="totalNumber">
</el-pagination>
```

```css
.el-pager li
{
    background: none !important;
}
.el-pagination .btn-next, .el-pagination .btn-prev
{
    background: none !important;
}
.el-pagination button:disabled
{
    background: none !important;
}
```

```javascript
data() {
    return {
        // for pagination
        // 当前页
        currentPage: 1,
        // 每页条数
        pageSize: 10,
        // 数据总数
        totalNumber: 0,
    }
},
methods: {
    // pagination
    // 页面条数修改时（没有设置这个功能）
    handleSizeChange(val) {
        // console.log(`每页 ${val} 条`);
    },
    handleCurrentChange(val) {
        console.log("val: " + val);
        request.get("/" + this.typeStr + "/" + this.currentPage + "/" + this.pageSize).then(res => {
            if (res.code === code.GET_OK) {
                // 根据res的返回结果进行赋值
                this.items = res.data.list;
                this.totalNumber = res.data.total;
                console.log("currentpage after: " + this.currentPage);
                console.log("total number: " + this.totalNumber);
            }
        }).catch(err => {
            console.log(err)
            this.$notify.error({
                title: '获取失败',
            });
        })
        // console.log(`当前页: ${val}`);
    },
}
```

后端：

```java
// BaseService interface
public interface BaseService<T> {
    public PageInfo<T> getAllByPage(int pageNum, int pageSize);
}
// BaseServiceImpl 
public abstract class BaseServiceImpl<T> implements BaseService<T> {
    @Override
    public PageInfo<T> getAllByPage(int pageNum, int pageSize) {
        PageHelper.startPage(pageNum, pageSize);
        List<T> baseList = baseMapper.getAll();
        return new PageInfo<T>(baseList);
    }
}
// BaseController
public abstract class BaseController<T> {
    @GetMapping("/{pageNum}/{pageSize}")
    public Result getAllByPage(@PathVariable Integer pageNum, @PathVariable Integer pageSize)
    {
        try
        {
            PageInfo<T> pageInfo = baseService.getAllByPage(pageNum, pageSize);
            Integer code = pageInfo != null ? Code.GET_OK : Code.GET_ERR;
            String msg = pageInfo != null ? "get succeeded" : "get failed";
            return new Result(code, pageInfo, msg);
        }
        catch (SystemException e)
        {
            throw new SystemException("unknown error occurred", Code.SYS_ERR);
        }
    }
}
```

### 搜索结果实时显示：

使用 `watch: ` 对 `<input>` 输入值进行监听。

```javascript
// 查询实时显示，watch监听
watch: {
    // 含输入的记得掐空格！！！
    searchInput(val) {
        if (val.trim() === '')
        {
            this.load();
            return;
        }
        // 延迟 0.2s 进行实时显示
        setTimeout(() => {
            request.get("/" + this.typeStr + "/" + val.trim() + 
                    "/" + this.currentPage + "/" + this.pageSize).then(res => {
                    if (res.code === code.GET_OK && res.data.total)
                    {
                        this.items = res.data.list;
                        this.totalNumber = res.data.total;
                    }
                    else 
                    {
                        this.items = [];
                        this.totalNumber = 0;
                        this.$notify.error({
                            title: message.FIND_ERR,
                        })
                    }
                }).catch(err => {
                    this.$notify.error({
                        title: message.REQUEST_ERR,
                    })
                })
        }, 200);
    }
},
```

### 浮动元素导致父元素高度塌陷：

添加以下css: 

```css
.father
{
}
.son
{
    float: left;
}
.father::after {
content:"";
/*清除浮动*/
clear: both;
/*确保该元素是一个块级元素*/
display: block;
}
```

### localStorage过期时间：

封装了`localStorage`，在存储的同时将时间存进去。

```javascript
// storage.js
let storage = {
    /*
    * set 存储方法
    * @ param {String}     key 键
    * @ param {String}     value 值，
    * @ param {String}     expired 过期时间，以毫秒为单位，非必须
    */
    set(key, val, expired) {
        let obj = {
            data: val,
            time: Date.now(),
            expired
        }
        localStorage.setItem(key, JSON.stringify(obj));
    },
    /*
    * get 获取方法
    * @ param {String}     key 键
    */
    get(key) {
        let val = localStorage.getItem(key);
        if (!val) {
            return val;
        }
        val = JSON.parse(val);
        if (Date.now() - val.time > val.expired) {
            localStorage.removeItem(key);
            return null
        }
        return val.data;
    },
    /*
    * remove 删除方法
    * @ param {String}     key 键
    */
    remove(key) {
        localStorage.removeItem(key);
    },
}
export default storage;
// main.js 中使用
import storage from './utils/storage.js'
Vue.prototype.$storage=storage;
// Vue组件中
this.$storage.set('item', item, 24 * 60 * 60);
```

### 解析 `Markdown` 字符串：

使用 `vue-markdown` 插件，并引入 `github-markdown-css` 文件。

```bash
npm install vue-loader vue-template-compiler -D
npm install --save vue-markdown
npm install github-markdown-css
```

```javascript
// main.js
import 'github-markdown-css/github-markdown.css'
```

```javascript
// vue组件内
import VueMarkdown from 'vue-markdown'

export default {
  components: {
    VueMarkdown // 注入组件
  },
  data () {
    return { 
      value: MarkdownData // value的值是要解析的markdown数据
    }
  }
}
```

```html
<!-- vue组件内 -->
<div class="markdown-body">
    <VueMarkdown :source="value"></VueMarkdown>
</div>
```

```css
/* 可以自定义样式 */
.markdown-body
{
    color: #2c3e50;
    font-size: 30px;
    background-color: rgba(252, 242, 241, 0);
}
```


### `mavon-editor` 图片上传、回显：  
  
`mavon-editor` 使用方法：[mavon-editor](https://github.com/hinesboy/mavonEditor)  

`mavon-editor` 图片上传：[pictures-upload](https://github.com/hinesboy/mavonEditor/blob/master/doc/cn/upload-images.md)

自定义 `imgAdd` 事件，后端设置上传接口；回显时设置 URL 磁盘映射，直接通过 URL 访问磁盘文件。

```javascript
// 前端
// 绑定@imgAdd event
$imgAdd(pos, $file){
    // 第一步. 将图片上传到服务器.
   var formdata = new FormData();
   formdata.append('file', $file);
   // request中的请求头是json格式，不方便
   this.$axios({
        url: this.$store.state.newsArticleImgBaseUrl + "upload",
        method: 'post',
        data: formdata,
        headers: { 'Content-Type': 'multipart/form-data' },
        timeout: 5000,
        // 解决前后session不一致问题
        withCredentials: true,
   }).then((res) => {
        // 第二步. 将返回的url替换到文本原位置![...](0) -> ![...](url)
        let result = JSON.parse(JSON.stringify(res));
        // console.log("result: " + result);
        if (result.data.code === code.UPLOAD_OK)
        {
            this.$refs.md.$img2Url(pos, 
                    this.$store.state.newsArticleImgBaseUrl +"download/" + result.data.data);
        }
   })
}
```

```java
// 后端
// 上传，Controller层
// 文章内图片上传
@PostMapping("/upload")
public Result upload(MultipartFile file)
{
    System.out.println("uploading...");
    // 获取的file是临时文件，需要转存

    File filePath = new File(basePath);
    // 若目录不存在，创建
    if (!filePath.exists())
    {
        filePath.mkdirs();
    }

    // 获取原始文件名后缀
    String postfix = Objects.requireNonNull(
                        file.getOriginalFilename()).substring(
                                (file.getOriginalFilename().lastIndexOf(".")));

    // 重新命名
    String fileName = GenerateMD5.encrypt(file.getOriginalFilename()) + postfix;

    // 转存文件
    try {
        file.transferTo(new File(basePath + fileName));
    }
    catch (IOException e) {
        throw new SystemException("upload failed", e, Code.SYS_ERR);
    }

    return new Result(Code.UPLOAD_OK, fileName, "upload succeeded");
}
// 下载，采用 URL 磁盘映射
// WebMvcConfig
@Configuration
public class WebMvcConfig extends WebMvcConfigurerAdapter {
    // basePath 通过 application.yml 注入，是磁盘上的绝对路径
    @Value("${atlantis.articlePicturesBasePath}")
    private String basePath;

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/newsArticles/download/**")
                .addResourceLocations("file:" + basePath);
    }
}
```

### `<div>` 元素内超过范围文字显示省略号: 

```css
.div 
{
    width: 60%;
    text-align: left;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}
```