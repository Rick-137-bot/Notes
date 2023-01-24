# 一、概念

SpringMVC是Spring的一个后续产品，是Spring的一个子项目
SpringMVC 是 Spring 为表述层开发提供的一整套完备的解决方案。

>注：三层架构分为表述层（或表示层）、业务逻辑层、数据访问层，表述层表示前台页面和后台servlet

>注：
/所匹配的请求可以是/login或.html或.js或.css方式的请求路径，但是/不能匹配.jsp请求路径的请求
/*则能够匹配所有请求，例如在使用过滤器时，若需要对所有请求进行过滤，就需要使用/*的写法

# 二、@RequesetMapping

将请求和处理请求的控制器方法关联起来，建立映射关系。

## 2.1 位置

``` java
@Controller
@RequestMapping("/test")
public class RequestMappingController {

    //此时请求映射所映射的请求的请求路径为：/test/testRequestMapping
    @RequestMapping("/testRequestMapping")
    public String testRequestMapping(){
        return "success";
    }

}
```

## 2.2 value属性

value是个列表表示该请求映射能够匹配多个请求地址所对应的请求

``` java
@RequestMapping(
        value = {"/testRequestMapping", "/test"}
)
public String testRequestMapping(){
    return "success";
}
```

## 2.3 method属性

通过请求的请求方式（get或post）匹配请求映射默认两者都接受。


> 注
1、对于处理指定请求方式的控制器方法，SpringMVC中提供了@RequestMapping的派生注解
处理get请求的映射-->@GetMapping
处理post请求的映射-->@PostMapping
处理put请求的映射-->@PutMapping
处理delete请求的映射-->@DeleteMapping
2、常用的请求方式有get，post，put，delete
但是目前浏览器只支持get和post，若在form表单提交时，为method设置了其他请求方式的字符串（put或delete），则按照默认的请求方式get处理
若要发送put和delete请求，则需要通过spring提供的过滤器HiddenHttpMethodFilter，在RESTful部分会讲到

## 2.4 params属性

通过请求的请求参数匹配请求映射

"param"：要求请求映射所匹配的请求必须携带param请求参数

"!param"：要求请求映射所匹配的请求必须不能携带param请求参数

"param=value"：要求请求映射所匹配的请求必须携带param请求参数且param=value

"param!=value"：要求请求映射所匹配的请求必须携带param请求参数但是param!=value

## 2.5 headers属性

通过请求的请求头信息匹配请求映射

"header"：要求请求映射所匹配的请求必须携带header请求头信息

"!header"：要求请求映射所匹配的请求必须不能携带header请求头信息

"header=value"：要求请求映射所匹配的请求必须携带header请求头信息且header=value

"header!=value"：要求请求映射所匹配的请求必须携带header请求头信息且header!=value

## 2.6 ant风格的路径

？：表示任意的单个字符

*：表示任意的0个或多个字符

\**：表示任意的一层或多层目录

注意：在使用\**时，只能使用/**/xxx的方式

## 2.7 路径中的占位符

原始方式：/deleteUser?id=1

rest方式：/deleteUser/1

SpringMVC路径中的占位符常用于RESTful风格中，当请求路径中将某些数据通过路径的方式传输到服务器中，就可以在相应的@RequestMapping注解的value属性中通过占位符{xxx}表示传输的数据，在通过@PathVariable注解，将占位符所表示的数据赋值给控制器方法的形参

``` html
<a th:href="@{/testRest/1/admin}">测试路径中的占位符-->/testRest</a><br>
```

``` java
@RequestMapping("/testRest/{id}/{username}")
public String testRest(@PathVariable("id") String id, @PathVariable("username") String username){
    System.out.println("id:"+id+",username:"+username);
    return "success";
}
//最终输出的内容为-->id:1,username:admin
```

# 三、SpringMVC获取请求参数

## 3.1 通过ServletAPI获取

与传统servlet相同通过HttpServletRequest

## 3.2 通过控制器方法的形参获取请求参数

``` html
<a th:href="@{/testParam(username='admin',password=123456)}">测试获取请求参数-->/testParam</a><br>
```

``` java
@RequestMapping("/testParam")
public String testParam(String username, String password){
    System.out.println("username:"+username+",password:"+password);
    return "success";
}
```

>注：
若请求所传输的请求参数中有多个同名的请求参数，此时可以在控制器方法的形参中设置字符串数组或者字符串类型的形参接收此请求参数
若使用字符串数组类型的形参，此参数的数组中包含了每一个数据
若使用字符串类型的形参，此参数的值为每个数据中间使用逗号拼接的结果

## 3.3 @RequestParam

将请求参数和控制器方法的形参创建映射关系

属性：
* value：指定为形参赋值的请求参数的参数名
* required：设置是否必须传输此请求参数，默认值为true
    * true:不传输且没有设置默认值就报错
    * false：不传输为null
* defaultValue：不管required属性值为true或false，当value所指定的请求参数没有传输或传输的值为""时，则使用默认值为形参赋值

## 3.4 @RequestHeader

将请求头信息和控制器方法的形参创建映射关系

共有三个属性：同上

## 3.5 @CookieValue

将cookie数据和控制器方法的形参创建映射关系

三个属性：同上

## 3.6 通过POJO获取请求参数

在控制器方法的形参位置设置一个实体类类型的形参，此时若浏览器传输的请求参数的参数名和实体类中的属性名一致，那么请求参数就会为此属性赋值

# 四、域对象共享数据

    



# 参考文献

[【尚硅谷】SpringMVC教程](https://www.bilibili.com/video/BV1Ry4y1574R)