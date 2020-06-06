## Thymeleaf是什么？

> Thymeleaf 提供spring标准方言和一个与 SpringMVC 完美集成的可选模块，可以快速的实现表单绑定、属性编辑器、国际化等功能。

> 使用:
>
> #{} 国际化
>
> ${} 取值
>
> *{} 取对象
>
> @{} 超链接对象

```java
@Controller
public class IndexController {
    @GetMapping("/user")
    public String userIndex(Model model){
        model.addAttribute("user",new User("男","张三"));
        return "user";
    }
}
```

```html
<!--使用${}获取文本的值--> 
<p th:text="${user.gender}+'['+${user.name}+']'"></p>
<p>[[${user.name}]]</p>
<!--男[张三]
    张三
-->

<!--
  我们获取用户的所有信息，分别展示。
  当数据量比较多的时候，频繁的写user.就会非常麻烦
  *{}
-->
 <h2 th:object="${user}">
    <p th:text="*{name}"></p>
    <p th:text="*{gender}"></p>
</h2>
<!--
   张三
   男
-->
<!--使用@{}获取超链接的地址值--> 
<link rel="stylesheet" href="../css/typo.css" th:href="@{/css/typo.css}">
```

### ognl表达式

```html
<h2 th:object="${user}">
    <p>FirstName: <span th:text="*{name.split(' ')[0]}">Jack</span>.</p>
    <p>LastName: <span th:text="*{name.split(' ')[1]}">Li</span>.</p>
</h2>
```

> ${user.name}` 可以写作`${user['name']}

### 逻辑处理

th:if

```html
<div th:if="true">
    你填的是true
</div>
<!--如果是true则显示这个标签与标签里的内容,如果是false则内容和标签都不显示-->
```

th:unless

> th:if ，两者的意思恰好相反。

th:switch

```html
<!--这里要使用两个指令：th:switch 和 th:case-->
<div th:switch="${user.role}">
  <p th:case="'admin'">用户是管理员</p>
  <p th:case="'manager'">用户是经理</p>
  <p th:case="*">用户是别的玩意</p>
</div>
<!--需要注意的是，一旦有一个th:case成立，其它的则不再判断。与java中的switch是一样的。
另外th:case="*"表示默认，放最后。-->
```



### 数据遍历

th:each=“user,index:${list}”

```html
<!--循环也是非常频繁使用的需求，我们使用th:each指令来完成：
    假如有用户的集合：users在Context中。
-->
<tr th:each="user,stat : ${users}">
    <th>stat</th>
    <td th:text="${user.name}">Onions</td>
    <td th:text="${user.age}">2.41</td>
</tr>
```



### 内置对象

> 我们在环境变量中添加日期类型对象

```java
@GetMapping("test3")
public String show3(Model model){
    model.addAttribute("today", new Date());
    return "hello3";
}
```

```java
<p>
  今天是: <span th:text="${#dates.format(today,'yyyy-MM-dd')}">2018-04-25</span>
</p>
```

