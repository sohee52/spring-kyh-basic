## 빈 스코프란?

- 스코프의 종류
    - 싱글톤
        - 기본 스코프
        - 스프링 컨테이너의 시작부터 종료까지 유지되는 가장 넓은 범위의 스코프
    - 프로토타입
        - 스프링 컨테이너가 프로토타입 **빈의 생성과 의존관계 주입, 초기화까지만 관여**하고 더는 관리하지 않는 매우 짧은 범위의 스코프
    - 웹 스코프
        - request
            - 웹 요청이 들어오고 나갈 때까지 유지되는 스코프
        - session
            - 웹 세션이 생성되고 종료될 때까지 유지되는 스코프
        - application
            - 웹의 서블릿 컨텍스트와 같은 범위로 유지되는 스코프
        - websocket
            - 웹 소켓과 동일한 생명주기를 가지는 스코프
- 빈 스코프 지정 방법
    - 컴포넌트 스캔 자동 등록

        ```java
        @Scope("prototype")
        @Component
        public class HelloBean {}
        ```

    - 빈 수동 등록

        ```java
        @Scope("prototype")
        @Bean
        PrototypeBean HelloBean() {
         return new HelloBean();
        }
        ```


## 프로토타입 스코프

- 싱글톤 스코프의 빈을 스프링 컨테이너에 요청하면?
    - 스프링 컨테이너는 본인이 관리하는 스프링 빈을 반환한다.
    - 이후에 스프링 컨테이너에 같은 요청이 와도 **같은 객체 인스턴스의 스프링 빈을 반환한다**
- 프로토타입 스코프의 빈을 스프링 컨테이너에 요청하면?
    - 스프링 컨테이너는 이 시점에 프로토타입 빈을 생성하고, 필요한 의존관계를 주입한다.
    - 스프링 컨테이너는 생성한 프로토타입 빈을 클라이언트에 반환한다.
    - 이후에 스프링 컨테이너에 같은 요청이 오면 **항상 새로운 프로토타입 빈을 생성해서 반환한다**
    - 클라이언트에 빈을 반환하고, **이후 스프링 컨테이너는 생성된 프로토타입 빈을 관리하지 않는다. 프로토타입 빈을 관리할 책임은 프로토타입 빈을 받은 클라이언트에 있다.**
    - 그래서 **@PreDestroy 같은 종료 메서드가 호출되지 않는다**

## 프로토타입 스코프 - 싱글톤 빈과 함께 사용 시 문제점

- 스프링은 일반적으로 싱글톤 빈을 사용하므로, 싱글톤 빈이 프로토타입 빈을 사용하게 된다. 이때 생기는 문제점은?
    - **싱글톤 빈은 생성시점에만 의존관계 주입을 받기 때문에, 프로토타입 빈이 새로 생성되기는 하지만, 싱글톤 빈과 함께 계속 유지되는 것이 문제다.**
    - 하지만 우리는 프로토타입 빈을 주입 시점에만 새로 생성하는 게 아니라, 사용할 때마다 새로 생성해서 사용하는 것을 원한다.

## 프로토타입 스코프 - 싱글톤 빈과 함께 사용 시 Provider로 문제 해결

- 싱글톤 빈과 프로토타입 빈을 함께 사용할 때, 어떻게 하면 사용할 때 마다 항상 새로운 프로토타입 빈을 생성할 수 있을까?
- Dependency Lookup(DL) 의존관계 조회(탐색)이란?
    - 의존관계를 외부에서 주입(DI) 받는게 아니라 이렇게 **직접 필요한 의존관계를 찾는 것**

### 스프링 컨테이너에 요청

- 싱글톤 빈이 프로토타입을 사용할 때마다 스프링 컨테이너에 새로 요청하는 방법의 문제점은?
    - 스프링의 애플리케이션 컨텍스트 전체를 주입받게 되면, 스프링 컨테이너에 종속적인 코드가 되고, 단위 테스트도 어려워진다.

    ```java
    package hello.core.scope;
    
    import jakarta.annotation.PostConstruct;
    import jakarta.annotation.PreDestroy;
    import org.junit.jupiter.api.Test;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.annotation.AnnotationConfigApplicationContext;
    import org.springframework.context.annotation.Scope;
    
    import static org.assertj.core.api.Assertions.assertThat;
    
    public class PrototypeProviderTest {
        @Test
        void providerTest() {
            AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(ClientBean.class, PrototypeBean.class);
            
            ClientBean clientBean1 = ac.getBean(ClientBean.class);
            int count1 = clientBean1.logic();
            assertThat(count1).isEqualTo(1);
            
            ClientBean clientBean2 = ac.getBean(ClientBean.class);
            int count2 = clientBean2.logic();
            assertThat(count2).isEqualTo(1);
        }
    
        static class ClientBean {
            @Autowired
            private ApplicationContext ac;
    
            public int logic() {
                PrototypeBean prototypeBean = ac.getBean(PrototypeBean.class); // 이렇게!
                prototypeBean.addCount();
                int count = prototypeBean.getCount();
                return count;
            }
        }
    
        @Scope("prototype")
        static class PrototypeBean {
            private int count = 0;
    
            public void addCount() {
                count++;
            }
    
            public int getCount() {
                return count;
            }
    
            @PostConstruct
            public void init() {
                System.out.println("PrototypeBean.init " + this);
            }
    
            @PreDestroy
            public void destroy() {
                System.out.println("PrototypeBean.destroy");
            }
        }
    }
    ```


### ObjectFactory, ObjectProvider

- ObjectProvider란?
    - 지정한 빈을 컨테이너에서 대신 찾아주는 DL 서비스를 제공하는 것
    - 과거에는 ObjectFactory 가 있었는데, 여기에 편의 기능을 추가해서 ObjectProvider 가 만들어졌다.
- ObjectProvider 코드

    ```java
       @Scope("singleton")
        static class ClientBean {
    
            @Autowired
            private ObjectProvider<PrototypeBean> prototypeBeanProvider;
    
            public int logic() {
                PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
                prototypeBean.addCount();
                int count = prototypeBean.getCount();
                return count;
            }
        }
    ```

- `prototypeBeanProvider.getObject()` 의 역할
    - 항상 새로운 프로토타입 빈을 생성한다.
    - **`ObjectProvider` 의 `getObject()` 를 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환한다. (DL)**
- ObjectProvider의 장점
    - 스프링이 제공하는 기능을 사용하지만, 기능이 단순하므로 **단위테스트를 만들거나 mock 코드를 만들기는 훨씬 쉬워진다**.
    - ObjectProvider 는 지금 딱 필요한 DL 정도의 기능만 제공한다.
- ObjectFactory vs ObjectProvider
    - ObjectFactory: 기능이 단순, 별도의 라이브러리 필요 없음, 스프링에 의존
    - ObjectProvider: ObjectFactory 상속, 옵션, 스트림 처리등 편의 기능이 많고, 별도의 라이브러리 필요 없음, 스프링에 의존
- ObjectProvider의 단점
    - **스프링 프레임워크에 의존한다.**

### JSR-330 Provider

- `javax.inject.Provider` 라는 JSR-330 자바 표준을 사용하는 방법
- 이 방법을 사용하려면 `jakarta.inject:jakarta.inject-api:2.0.1`  라이브러리를 gradle에 추가해야 한다.
- 코드

    ```java
        @Scope("singleton")
        static class ClientBean {
    
            @Autowired
            private Provider<PrototypeBean> provider;
    
            public int logic() {
                PrototypeBean prototypeBean = provider.get();
                prototypeBean.addCount();
                int count = prototypeBean.getCount();
                return count;
            }
        }
    ```

- `provider.get()` 의 역할
    - 항상 새로운 프로토타입 빈이 생성되는 것을 확인할 수 있다.
    - **`provider` 의 `get()` 을 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환한다. (DL)**
- JSR-330 Provider의 특징(장점)
    - **별도의 라이브러리가 필요하다.**
    - **자바 표준이므로 스프링이 아닌 다른 컨테이너에서도 사용할 수 있다.**
    - get() 메서드 하나로 기능이 매우 단순하다.
    - 자바 표준이고, 기능이 단순하므로 단위테스트를 만들거나 mock 코드를 만들기는 훨씬 쉬워진다.
    - Provider 는 지금 딱 필요한 DL 정도의 기능만 제공한다.

### 실무

- 프로토타입 빈을 언제 사용할까?
    - 매번 사용할 때 마다 의존관계 주입이 완료된 새로운 객체가 필요하면 사용하면 된다
- 실무에선?
    - 실무에서 웹 애플리케이션을 개발해보면, 싱글톤 빈으로 대부분의 문제를 해결할 수 있기 때문에 프로토타입 빈을 직접적으로 사용하는 일은 매우 드물다.
- ObjectProvider , JSR330 Provider 등은 언제 사용할까?
    - 프로토타입 뿐만 아니라 DL이 필요한 경우는 언제든지 사용할 수 있다.
- ObjectProvider , JSR330 Provider 둘 중 무엇을 사용해야 할까?
    - **ObjectProvider는 DL을 위한 편의 기능을 많이 제공해주고 스프링 외에 별도의 의존관계 추가가 필요 없기 때문에 편리하다.**
    - **만약(정말 그럴일은 거의 없겠지만) 코드를 스프링이 아닌 다른 컨테이너에서도 사용할 수 있어야 한다면 JSR-330 Provider를 사용해야한다.**
    - 스프링을 사용하다 보면 이 기능 뿐만 아니라 다른 기능들도 자바 표준과 스프링이 제공하는 기능이 겹칠때가 많이 있다. 대부분 스프링이 더 다양하고 편리한 기능을 제공해주기 때문에, 특별히 다른 컨테이너를 사용할 일이 없다면, 스프링이 제공하는 기능을 사용하면 된다.

## 웹 스코프

- 웹 스코프의 특징
    - 웹 스코프는 웹 환경에서만 동작한다.
    - 웹 스코프는 프로토타입과 다르게 스프링이 해당 스코프의 종료시점까지 관리한다. 따라서 종료 메서드가 호출된다.
- 웹 스코프의 종류
    - request
        - HTTP 요청 하나가 들어오고 나갈 때 까지 유지되는 스코프
        - 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고, 관리된다.
    - session
        - HTTP Session과 동일한 생명주기를 가지는 스코프
    - application
        - 서블릿 컨텍스트( `ServletContext` )와 동일한 생명주기를 가지는 스코프
    - websocket
        - 웹 소켓과 동일한 생명주기를 가지는 스코프

## request 스코프 예제 만들기

- 여러 HTTP 요청이 동시에 오면 왜 로그 구분이 어려울까요?
    - 콘솔에 단순 문자열만 남기면, 같은 시점에 찍힌 로그가 어느 요청에서 발생했는지 식별할 수 없다.
    - **각 요청마다 고유한 정보(예: UUID·요청 URL)가 함께 기록돼야 구분이 가능하다.**
- Request 스코프란 무엇이고,  문제를 어떻게 해결하나요?
    - **`@Scope("request")` 빈은 HTTP 요청당 하나씩 생성·소멸되는 스프링 빈이다.**
    - **요청마다 고유한 UUID와 URL을 보관해 로그에 붙이면, 콘솔에서도 “같은 요청 흐름”을 손쉽게 추적할 수 있습니다.**
- `MyLogger` 클래스는 어떤 역할을 하나요?
    - `@Component + @Scope("request")`: 요청마다 별도 인스턴스.
    - `@PostConstruct`: UUID 생성 후 `[UUID] request scope bean create` 로그 출력.
    - `setRequestURL(String)`: 컨트롤러(혹은 필터/인터셉터)가 URL 주입.
    - `log(String)`: `[UUID][URL] 메시지` 형식으로 로그 출력.
    - `@PreDestroy`: 요청 종료 시 `[UUID] request scope bean close` 로그 출력.
- 컨트롤러·서비스에서는 어떻게 사용하나요?
    - **컨트롤러**: `request.getRequestURL()`로 URL을 얻어 `myLogger.setRequestURL(...)` 호출 후 `myLogger.log("controller test")` 출력.
    - **서비스**: 비즈니스 로직 수행 후 `myLogger.log("service id = ...")` 호출
- 애플리케이션 부팅 시점에 “Scope 'request' is not active” 예외가 나는 이유는?
    - 싱글톤 빈 주입 단계에서 스프링이 `MyLogger`를 바로 주입하려 하지만, 아직 실제 HTTP 요청이 없기 때문에 request 스코프가 활성화되지 않아 예외가 발생한다.
    - **Request 스코프 빈은 실제 고객의 요청이 와야 생성이 되기에 애플리케이션 실행 시점에 오류가 발생할 수 있다**

## 스코프와 Provider

- `ObjectProvider`로 이 문제를 어떻게 해결하나요?
    1. 빈 대신 `ObjectProvider<MyLogger>`를 주입한다.
    2. 실제 요청 처리 메서드 안에서 `myLoggerProvider.getObject()`를 호출해 **그 순간** 빈을 조회한다.
    3. 이때는 이미 요청이 진행 중이므로 request 스코프 빈이 정상적으로 생성돼 예외가 발생하지 않는다.
- 같은 HTTP 요청 안에서 컨트롤러와 서비스가 `getObject()`를 각각 호출하면 서로 다른 인스턴스를 받나요?
    - No. 같은 요청 범위이므로 **동일한 `MyLogger` 인스턴스**가 반환된다.
- URL 같은 웹 정보를 서비스 계층까지 넘기지 않으면 어떤 이점이 있나요?
    - 서비스 계층이 웹 기술(HTTP, URL 등)에 **종속되지 않고 순수**하게 유지된다.
    - 유지보수·테스트가 쉬워지고 계층 간 역할이 명확해집니다.
- 하지만 `ObjectProvider`보다 더 간단하게 코드를 칠 수 있는 녀석이 있으니.. 그건 바로 프록시!

## 스코프와 프록시

- 프록시 방식을 사용하려면 어떻게 코드를 적어야 하나요?

    ```java
    @Component
    @Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
    public class MyLogger {
    }
    ```

    - `proxyMode = ScopedProxyMode.TARGET_CLASS` 를 추가해주자.
        - 적용 대상이 인터페이스가 아닌 **클래스면 `TARGET_CLASS` 를 선택**
        - 적용 대상이 **인터페이스면 `INTERFACES` 를 선택**
    - **이렇게 하면 MyLogger의 가짜 프록시 클래스를 만들어두고 HTTP request와 상관 없이 가짜 프록시 클래스를 다른 빈에 미리 주입해 둘 수 있다.**
- 프록시 동작 원리
    - CGLIB라는 라이브러리로 내 클래스를 상속 받은 가짜 프록시 객체를 만들어서 주입한다.
    - 이 가짜 프록시 객체는 실제 요청이 오면 그때 내부에서 실제 빈을 요청하는 위임 로직이 들어있다.
    - 가짜 프록시 객체는 실제 request scope와는 관계가 없다. 그냥 가짜이고, 내부에 단순한 위임 로직만 있고, 싱글톤 처럼 동작한다.
- 주의할 점
    - 마치 싱글톤을 사용하는 것 같지만 다르게 동작하기 때문에 결국 주의해서 사용해야 한다.
    - 이런 특별한 scope는 꼭 필요한 곳에만 최소화해서 사용하자, 무분별하게 사용하면 유지보수하기 어려워진다.