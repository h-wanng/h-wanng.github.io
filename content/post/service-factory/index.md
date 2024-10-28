+++
author = "H. Wang"
title = "Service Factory"
date = "2024-09-18"
description = "Spring依赖注入和策略模式实现Service工厂"
tags = [
    "Java", "Spring"
]
categories = [
    "Java"
]

image = ""
+++

Spring中常出现对于Service的多实现来处理不同业务。例如有一个告警上报的业务场景，需要处理不同类型的告警。又如通过继承抽象类的方式处理不同类型日志类型内容的生成。

可以使用`ApplicationContext`或三方工具`SpringUtil.getBean(Xxx.class);`获取Bean。也可使用Spring的依赖注入和策略模式来实现。

```java
@Component
public class MyServiceFactory {

    private final Map<String, MyService> serviceMap;

    @Autowired
    public MyServiceFactory(List<MyService> services) {
        serviceMap = services.stream().collect(Collectors.toMap(
                service -> service.getClass().getSimpleName().replace("MyServiceImpl", "").toLowerCase(),
                service -> service
        ));
    }

    public MyService getService(String type) {
        MyService service = serviceMap.get(type.toLowerCase());
        if (service == null) {
            throw new IllegalArgumentException("Unknown service type: " + type);
        }
        return service;
    }
}
```

使用工厂类：

```Java
@RestController
@RequestMapping("/api")
public class MyController {

    private final MyServiceFactory myServiceFactory;

    @Autowired
    public MyController(MyServiceFactory myServiceFactory) {
        this.myServiceFactory = myServiceFactory;
    }

    @GetMapping("/process")
    public ResponseEntity<String> process(@RequestParam String type) {
        MyService myService = myServiceFactory.getService(type);
        myService.process();
        return ResponseEntity.ok("Processing done by " + type);
    }
}
```

