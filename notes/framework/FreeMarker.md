# 一、什么是FreeMarker

 FreeMarker 是一款 *模板引擎*： 即一种基于模板和要改变的数据， 并用来生成输出文本(HTML网页，电子邮件，配置文件，源代码等)的通用工具。 它不是面向最终用户的，而是一个Java类库，是一款程序员可以嵌入他们所开发产品的组件。

# 二、Spring Boot 整合 FreeMarker

## 1、引入依赖

在你的 `pom.xml` 中添加 Starter，Spring Boot 会自动配置 `FreeMarkerConfigurer` 和 `ViewResolver`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```

## 2、配置文件

关键配置在于指定模板路径和后缀名。开发阶段务必关闭缓存，以便实时看到修改效果

```yml
spring:
  freemarker:
    template-loader-path: classpath:/templates/   # 模板存放位置
    suffix: .ftl                                  # 文件后缀
    cache: false                                  # 开发时关闭缓存
    charset: UTF-8
    content-type: text/html
```

## 3、控制层编写

FreeMarker 与 Spring Boot 的集成非常丝滑。你只需要在 `Controller` 中返回视图名称（String）或 `ModelAndView`，并将数据放入 `Model` 中即可。

```java
@Controller // 注意是 @Controller 不是 @RestController
public class UserController {

    @GetMapping("/user")
    public String getUser(Model model) {
        model.addAttribute("name", "张三");
        model.addAttribute("age", 25);
        return "userProfile"; // 对应 templates/userProfile.ftl
    }
}
```

# 三、核心语法与常用指令

## 1、插值与变量操作

- **输出变量**：`${user.name}` 或 `user.name`。
- **判空与默认值**：这是实际开发中最常用的功能，用于避免页面报错。
  - `${name!''}` ：如果 `name` 不存在，则显示空字符串。
  - `${name!'默认值'}` ：设置默认值。
- **exists/if_exists 处理**：

```xml
<#if user??>   <#-- 判断user变量是否存在且不为null -->
  用户存在
</#if>
```

##  2、逻辑与循环指令

- **条件判断 (`<#if>`)** ：支持 `?string` 方法将布尔值转为字符串。

  ```xml
  <#if score >= 60>
      <span class="pass">及格</span>
  <#else>
      <span class="fail">不及格</span>
  </#if>
  ```

- **列表循环 (`<#list>`)** ：`item_index` 是内置的索引（从0开始），`item_has_next` 判断是否有下一个。

```xml
<#list userList as user>
    <tr>
        <td>${user_index + 1}行</td>
        <td>${user.name}</td>
    </tr>
</#list>
```

- **宏定义 (`<#macro>`)** ：用于复用页面片段（如分页组件），类似函数。

```xml
<#macro sayHello name>
    <h2>Hello ${name}!</h2>
</#macro>
<@sayHello name="张三" />
```

## 3、内置函数

内建函数是 FreeMarker 的一大亮点，能直接在模板中处理数据。

- **字符串**：`${user.name?upper_case}` (转大写)、`${user.name?cap_first}` (首字母大写)。
- **日期**：`${createTime?string("yyyy-MM-dd HH:mm:ss")}`。
- **集合**：`${myList?size}` (获取长度)、`${myList?join(", ")}` (以逗号拼接)。

# 四、常见问题

| 考察维度          | 核心要点与面试回答话术                                       |
| :---------------- | :----------------------------------------------------------- |
| **与 JSP 的区别** | JSP 本质是 Servlet，包含隐式对象，可以在页面写 Java 代码，破坏 MVC 结构；FreeMarker **严格分离**视图与业务逻辑，加载速度更快，且不依赖 Servlet 容器，适合生成静态化文件。 |
| **工作原理**      | 后端准备数据填充到 `Map` 中，FreeMarker 读取 `.ftl` 模板文件，结合数据模型通过 `process` 方法渲染，最终输出 HTML 等文本内容到浏览器。 |
| **空值处理**      | 使用 **`!`** 运算符。例如 `${user!}` 或 `${user!'默认值'}`。如果不处理，变量不存在时 FreeMarker 会直接抛出 `TemplateException`，这是新手常犯的错误。 |
| **性能优化**      | 核心在于**开启缓存** (`cache=true`)，避免每次请求都重新解析模板。同时，不要在模板中进行过于复杂的计算或数据库查询，保持模板的逻辑简单性。 |