---
layout: post
title: "Spring 커맨드라인 기반 예제 from Toby"
date: "2015-09-23 21:20"
tags: [docker, error]
---

최근 각종 개발자 방송에 유행이다. 나프다, 코코티비 등 뿐만 아니라 웹비나도 같이 유행이다.

**그런데** 최근에 토비의 스프링을 저술하신 Toby님 께서 라이브코딩 방송을 하였다.
관련해서 해당 내용을 복기하는 차원에서 포스트를 해본다.

- 간단하게 내용을 이야기 하자면 스프링 어플리케이션 초기화(자원할당), 마무리(자원해제) 에 대한 고찰이다.

> [토비님 방송 링크][TobyTVLink]

## Step 1. Simple
```Java
@SpringBootApplication
public class Springcl2Application {
  public static void main(String[] args) {
    SpringApplication.run(Springcl2Application.class, args);

    System.out.println("main()");
  }
}
```

단순하게 `main()`을 출력하고 마무리.

- 참고로 `@SpringBootApplication` 는 각종 [메타 어너테이션][MetaAnnotation]이 선언되어 있다.
- 기본적으로 `@Target(ElementType.TYPE)` 으로 선언되어 있으면 `ElementTypeANNOTATION_TYPE` 에서도 사용할 수 있기 때문에 일반 어너테이션 이면서 메타 어너테이션이 될 수 있다.
- 메타 어너테이션으로만 사용하고 싶으면 자바 기본 메타 어너테이션 처럼 `@Target(ElementType.ANNOTATION_TYPE)` 으로 선언하면 된다.

```Java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Configuration
@EnableAutoConfiguration
@ComponentScan
public @interface SpringBootApplication {
  ...
}
```


## Step 2. Implementation CommandLineRunner
```Java
@SpringBootApplication
public class Springcl2Application implements CommandLineRunner {
  public static void main(String[] args) {
    SpringApplication.run(Springcl2Application.class, args);

    System.out.println("main()");
  }

  @Override
  public void run(String... args) throws Exception {
    System.out.println("run()");
  }
}
```

1. `run()` 출력
2. `main()` 출력

## Step 3. CommandLineRunner @Bean 등록 with Lamda Expression
```Java
@SpringBootApplication
public class Springcl2Application {
  public static void main(String[] args) {
    SpringApplication.run(Springcl2Application.class, args);

    System.out.println("main()");
  }

  @Bean
  public CommandLineRunner runner() {
    return (a) -> {
      System.out.println("runner()");
    };
  }
}
```

1. `runner()` 출력
2. `main()` 출력

## Step 4. CommandLineRunner @Component 등록
```Java
@SpringBootApplication
public class Springcl2Application {
  public static void main(String[] args) {
    SpringApplication.run(Springcl2Application.class, args);

    System.out.println("main()");
  }

  @Component
  public static class MyRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
      System.out.println("MyRunner#run()");
    }
  }
}
```

1. `runnerMyRunner#run()` 출력
2. `main()` 출력

## Step 5. InitializingBean @Bean 등록 with Lamda Expression
```Java
@SpringBootApplication
public class Springcl2Application {
  public static void main(String[] args) {
    SpringApplication.run(Springcl2Application.class, args);

    System.out.println("main()");
  }

    @Bean
    public InitializingBean runner() throws Exception {
      return () -> {
        System.out.println("afterPropertiesSet()");
      };
    }
}
```

1. `afterPropertiesSet()` 출력
2. `main()` 출력

## Step 6. try-with-resources
```Java
@SpringBootApplication
public class Springcl2Application {
  public static void main(String[] args) {
    try (ConfigurableApplicationContext ac = SpringApplication.run(Springcl2Application.class, args)) {
      System.out.println("Something");
    }
    System.out.println("main()");
  }
}
```

1. `Something` 출력
2. `main()` 출력

## Step 7. try-with-resources with Lamda Expression
```Java
@SpringBootApplication
public class Springcl2Application {
  public static void main(String[] args) throws Exception {
    ConfigurableApplicationContext ac = SpringApplication.run(Springcl2Application.class, args);
    try (AutoCloseable closable = () -> { System.out.println("Auto close"); })
      System.out.println("Something");
    }
    System.out.println("main()");
  }
}
```

1. `Something` 출력
2. `Auto close` 출력
3. `main()` 출력

## Step 8. Runtime.addShutdownHook
```Java
@SpringBootApplication
public class Springcl2Application {
  public static void main(String[] args) {
    Runtime.getRuntime().addShutdownHook(new Thread() {
      @Override
      public void run() {
        System.out.println("shutdown");
      }
    });
    ConfigurableApplicationContext ac = SpringApplication.run(Springcl2Application.class, args);
    try (AutoCloseable closable = () -> { System.out.println("Auto close"); }) {
      System.out.println("Something");
    }
    System.out.println("main()");
  }
}
```

1. `Something` 출력
2. `Auto close` 출력
3. `main()` 출력
4. `shutdown` 출력

> [Runtime.addShutdownHook 참조 링크][RuntimeAddShutdownHook]

## Conclusion

스프링의 **라이프사이클** 과 자바에서 제공하는 라이브러리 등을 이용해서 스프링 어플리케이션 구동 시 각종 초기화, 마무리 작업을 다양한 방법으로 제어할 수 있다.


[TobyTVLink]: http://youtu.be/dnCf2-XYXL8
[MetaAnnotation]: https://en.wikibooks.org/wiki/Java_Programming/Annotations/Meta-Annotations
[RuntimeAddShutdownHook]: http://hellotojavaworld.blogspot.com.au/2010/11/runtimeaddshutdownhook.html?m=1
