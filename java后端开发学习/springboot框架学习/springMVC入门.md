# springMVC入门

[toc]

## MVC定义

MVC框架，即模型model、视图view、控制器controller。MVC是最常用的行业标准的Web开发框架。

> **模型:** 模型组件对应于所有相关的逻辑，用户和数据。这可以表示正在被视图和控制器组件或任何其他业务逻辑相关数据之间传输的数据。例如，一个客户对象将从数据库中检索的客户信息，操纵，并更新数据返回到数据库或者用它来呈现数据。
>
> **视图:** 视图组件用于应用程序的所有用户界面逻辑。例如，客户视图将包括所有的UI组件，例如文本框，下拉菜单等，最终与用户交互。
>
> **控制器:** 控制器充当Model和View组件之间的接口，用以处理所有的业务逻辑和传入的请求， 使用模型部件操纵数据以及与视图交互以显示最终的输出。**控制器不做任何的数据处理**。例如，客户控制器将处理所有的交互和输入来自客户查看和使用客户模型更新数据库。相同的控制器将用于查看客户数据。

## 注解

### @Controller

`@Controller`添加到类上，表示这个类是Controller对象。属性如下：

- `name` 属性：该Controller对象的Bean名字。允许空。

`@RestController`注解是`@Controller`和`@ResponseBody`的组合注解，直接使用接口方法的返回结果，经过JSON/XML等序列方式，最终返回。也就是说，无需使用InternalResourceViewResolver解析视图，返回HTML结果。

### @RequestMapping

`@RequestMapping`注解，添加在类或方法上，表示该类/方法对应接口的配置信息。

常用属性如下：

- `path`属性：接口路径。`[]`数组，可以填多个接口路径。
- `values`属性：和`path`属性相同，是它的别名。
- `method`属性：请求方法类型，可以填`GET`、`POST`、`PUT`、`DELETE`等。`[]`数组，可以填多个请求方法。**如果为空，表示匹配所有请求方法**。

不常用属性如下：

- `name`属性：接口名。
- `params`属性：指定请求中必须包含某些参数值
- `headers`：指定请求中必须包含某些指定的header值
- `consumes`：指定请求的提交内容类型（Content-Type），例如application/json，text/html
- `produces`：指定返回的内容类型，仅当request请求头中的（Accept）类型中包含该指定类型才返回



