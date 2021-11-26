---
title: SpringBoot下处理跨域请求
date: 2021-09-12 15:37:40
tags: [Program, SpringBoot, Java]
categories: Program
cover: https://z3.ax1x.com/2021/10/08/59y6SK.jpg
copyright_author: B1ackSand
copyright_author_href: https://github.com/B1ackSand
copyright_url: https://blacksand.top/2021/09/12/SpringBoot下处理跨域请求
copyright_info: 此文章版权归B1ackSand所有，如有转载，请注明来自原作者
---

## 一、前言

### 什么是跨域请求？

`Url`的一般格式：

```
协议 + 域名（子域名 + 主域名） + 端口号 + 资源地址
```

示例：

```
https://www.blacksand.top:8080/say/Hello 是由

https + www + blacksand.top + 8080 + say/Hello 组成
```

 组成。



```
只要协议，子域名，主域名，端口号这四项组成部分中有一项不同，就可以认为是不同的域，不同的域之间互相访问资源，就被称之为跨域。
```



## 二、Cors跨域

### Cors是什么

```
CORS全称为Cross Origin Resource Sharing（跨域资源共享）, 每一个页面需要返回一个名为Access-Control-Allow-Origin的http头来允许外域的站点访问，你可以仅仅暴露有限的资源和有限的外域站点访问。
```

 我们可以理解为：如果一个请求需要允许跨域访问，则需要在`http`头中设置

```
Access-Control-Allow-Origin来决定需要允许哪些站点来访问。如假设需要允许https://www.blacksand.top这个站点的请求跨域，则可以设置：

Access-Control-Allow-Origin:https://www.blacksand.top
```

### 如何解决Cors跨域

#### 1. 用`@CrossOrigin`注解

在`Controller`上使用`@CrossOrigin`注解使得该类下的所有接口都可以通过跨域访问。

```java
@CrossOrigin
//@CrossOrigin("https://blacksand.top") //只有指定域名可以访问该类下所有接口
@RestController
@RequestMapping("/question")
@Api(tags = "题目控制器")
public class QuestionController {

    @Resource
    private IQuestionService questionService;

    /*
     * 测试接口
     * @return "hello"
     * */

    @ApiOperation("测试接口")
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
```

这里指定当前的`QuestionController`中所有的方法可以处理`https://blacksand.top`域上的请求,这里可以在访问`https://blacksand.top`时，打开浏览器的Console中测试一下

```js
var token= "LtSFVqKxvpS1nPARxS2lpUs2Q2IpGstidMrS8zMhNV3rT7RKnhLN6d2FFirkVEzVIeexgEHgI/PtnynGqjZlyGkJa4+zYIXxtDMoK/N+AB6wtsskYXereH3AR8kWErwIRvx+UOFveH3dgmdw1347SYjbL/ilGKX5xkoZCbfb1f0=,LZkg22zbNsUoHAgAUapeBn541X5OHUK7rLVNHsHWDM/BA4DCIP1f/3Bnu4GAElQU6cds/0fg9Li5cSPHe8pyhr1Ii/TNcUYxqHMf9bHyD6ugwOFTfvlmtp6RDopVrpG24RSjJbWy2kUOOjjk5uv6FUTmbrSTVoBEzAXYKZMM2m4=,R4QeD2psvrTr8tkBTjnnfUBw+YR4di+GToGjWYeR7qZk9hldUVLlZUsEEPWjtBpz+UURVmplIn5WM9Ge29ft5aS4oKDdPlIH8kWNIs9Y3r9TgH3MnSUTGrgayaNniY9Ji5wNZiZ9cE2CFzlxoyuZxOcSVfOxUw70ty0ukLVM/78=";
var xhr = new XMLHttpRequest();
xhr.open('GET', 'http://127.0.0.1:8080/answer/hello');
xhr.setRequestHeader("x-access-token",token);
xhr.send(null);
xhr.onload = function(e) {
    var xhr = e.target;
    console.log(xhr.responseText);
}
```

如果Console返回Hello，说明跨域成功。



#### 2. CORS全局配置-实现`WebMvcConfigurer`

- 新建跨域配置类：`CorsConfig.java`:

  ```java
  /**
   * 跨域配置
   */
  @Configuration
  public class CorsConfig implements WebMvcConfigurer {
  
      @Bean
      public WebMvcConfigurer corsConfigurer()
      {
          return new WebMvcConfigurer() {
              @Override
              public void addCorsMappings(CorsRegistry registry) {
                  registry.addMapping("/**").
                          allowedOrigins("https://www.blacksand.top"). //允许跨域的域名，可以用*表示允许任何域名使用
                          allowedMethods("*"). //允许任何方法（post、get等）
                          allowedHeaders("*"). //允许任何请求头
                          allowCredentials(true). //带上cookie信息，如允许任何域名跨域则需为false
                          exposedHeaders(HttpHeaders.SET_COOKIE).maxAge(3600L); //maxAge(3600)表明在3600秒内，不需要再发送预检验请求，可以缓存该结果
              }
          };
      }
  }
  ```

+ 同样进行上面的测试，查看是否返回Hello。



#### 3. 拦截器Filter实现

通过实现`Fiter`接口在请求中添加一些`Header`来解决跨域的问题。

```java
@Component
public class CorsFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletResponse res = (HttpServletResponse) response;
        res.addHeader("Access-Control-Allow-Credentials", "true");
        res.addHeader("Access-Control-Allow-Origin", "*");
        res.addHeader("Access-Control-Allow-Methods", "GET, POST, DELETE, PUT");
        res.addHeader("Access-Control-Allow-Headers", "Content-Type,X-CAF-Authorization-Token,sessionToken,X-TOKEN");
        if (((HttpServletRequest) request).getMethod().equals("OPTIONS")) {
            response.getWriter().println("ok");
            return;
        }
        chain.doFilter(request, response);
    }
    @Override
    public void destroy() {
    }
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }
}
```

至此,跨域请求就完成了。



## 三、参考链接

```
https://www.cnblogs.com/zhaosq/p/11410682.html
```



