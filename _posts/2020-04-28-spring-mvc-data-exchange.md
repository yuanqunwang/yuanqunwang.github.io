---
title: Spring MVC 与客户端的交换数据
tags: [SpringMVC, rest]
---

## `HttpMessageConverter<T>`

`HttpMessageConverter<T>`是Spring中一个重要的接口，能够将请求信息转换为`T`实例对象，或将`T`实例对象输出为响应信息。该接口中声明的方法如下：

```java
public interface HttpMessageConverter<T> {
  // 转换器是否可以将请求信息（类型为meidaType）转换为clazz类型对象
  Boolean canRead(Class<?> clazz, MediaType mediaType);
  // 转换器是否可以将clazz类型对象写到响应信息
  Boolean canWrite(Class<?> clazz, MedizType mediaType);
  // 转换器支持的媒体类型
  List<MediaType> getSupportedMediaType();
  // 将请求信息转换为T实例对象
  T read(Class<? extends T> clazz, HttpInputMessage inputMessage);
  // 将T实例对象写入到响应信息
  void write(T t, MediaType mediaType, HttpOutputMessage outputMessage);
}
```

根据上述接口方法签名可知，转换器在将请求信息转换为Java对象，或将Java对象写入到响应信息中，需要如下两个信息：

* `MediaType`，通过请求的Content-Type或Accept属性获得。
* `Class<?>`，根据方法签名获得，或者通过`HttpEntity`或`ResponseEntity`的泛型获得。

`DispatcherServlet`默认安装的`RequestMappingHandlerAdapter`是`HandlerAdapter`的实现类，`RequestMappingHandlerAdapter`内部使用`HttpMessageConverter<T>`，将请求信息转换为Java对象，或将Java对象写入到响应信息。

SpringMVC中默认实现了多种转换器，根据请求的属性和方法签名，自动选择合适的转换器进行转换。这是一种策略模式。

## 数据准备

```java
public class Car {
    private String brand;
    private String series;
    private double price;
}
```

##  `@RequestBody` 和 `@ResponseBody`

### 将请求信息转换为 JavaBean

```java
    @RequestMapping("/request1")
    @ResponseBody
    public Car request1(@RequestBody Car car) {
        car.setPrice(300.0);
        return car;
    }
```

```shell
$ curl -X post --data '{"brand":"BMW","series":"520","price":50000.0}' --header 'Content-Type:application/json' localhost:8080/request1
// 返回结果
$ {"brand":"BMW","series":"520","price":300.0}
```

`@RequestBody`和`@ResponseBody`不一定需要同时使用。

### 将请求信息转换为Map

```java
    @RequestMapping("/request2")
    @ResponseBody
    public void request2(@RequestBody String car) throws JsonProcessingException {
        ObjectMapper mapper = new ObjectMapper();
        Map<String, Object> carMap = 
          mapper.readValue(car, new TypeReference<Map<String, Object>>() {});
        System.out.println(carMap);
    }
```

在终端输入上述 `curl` 命令后，服务端将打印出结果：

```shell
{brand=BMW, series=520, price=50000.0}
```

## `HttpEntity<T>` 与 `ResponseEntity<T>`

`@RequestBody`可以获得请求报文体中的内容，这点`HttpEntity<T>`与之相似，同时`HttpEntity<T>`还可以获得报文头的信息。

```java
@RequestMapping("/request3")
public void request3(HttpEntity<Car> entity) {
  Car car = entity.getBody();
}
```

`ResponseEntity<T>`用法

```java
@RequestMapping("/request4")
public ResponseEntity<Car> request() {
  Car car = new Car();
  // setting...
  ResponseEntity<Car> entity = 
    new ResponseEntity<>(car, HttpStatus.Ok);
  return entity;
}
```

## `@RestController`

`@Restcontroller` = `@ResponseBody` + `@Controller`

该注解支持 REST 风格应用的开发。