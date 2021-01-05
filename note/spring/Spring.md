# 注解
## @ControllerAdvice
// 可指定包前缀(basePackages = "com.fth.controlelr")
作用:
1. 全局异常处理
```java
@ControllerAdvice
@Order(-1) // 存在其他优先级的异常处理类时 可以选择使用。
public class GlobalExceptionHandler {

    @ExceptionHandler(NullPointerException.class)
    @ResponseBody 
    public String nullException(NullPointerException e){
        return e.getMessage();
    }
}
```

2. 全局数据绑定
全局数据绑定功能可以用来做一些初始化的数据操作，我们可以将一些公共的数据定义在添加了 @ControllerAdvice 注解的类中，这样，在每一个 Controller 的接口中，就都能够访问导致这些数据.
```java
@ControllerAdvice
@RestController
public class GlobalDataBind {

    @ModelAttribute(name = "user")
    public Map<String,String> user(){
        Map<String,String> user = new HashMap<>();
        user.put("age","18");
        user.put("role","admin");
        return user;
    }

	@RequestMapping("/binddata")
    @ResponseBody
    public Object bindData(Model model){
        return model.getAttribute("user");
    }
}
```

3. 全局数据预处理 
请求参数中，包含两个对象，两个对象中有部分属性值名称一样。
```java

@ControllerAdvice
@RestController
public class DataHandler {

    @InitBinder("person")
    public void person(WebDataBinder webDataBinder){
        webDataBinder.setFieldDefaultPrefix("person.");
    }

    @InitBinder("pet")
    public void pet(WebDataBinder webDataBinder){
        webDataBinder.setFieldDefaultPrefix("pet.");
    }

	@RequestMapping("/personAndPet")
	public Map<String,String> personAndPet(@ModelAttribute("person") Person person,@ModelAttribute("pet") Pet pet){
	    Map<String,String> map = new HashMap<>();
	    map.put("person",person.getName());
	    map.put("pet",pet.getName());
	    return map;
	}
}

http://localhost:8084/personAndPet?person.name=zs&pet.name=xb

```

