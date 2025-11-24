## 다양한 의존관계 주입 방법

- 다양한 의존관계 주입 방법
    - 생성자 주입
        - = 생성자를 통해서 의존 관계를 주입 받는 방법

        ```java
        @Component
        public class OrderServiceImpl implements OrderService{
        
            private final MemberRepository memberRepository;
            private final DiscountPolicy discountPolicy;
        
            @Autowired
            public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
                this.memberRepository = memberRepository;
                this.discountPolicy = discountPolicy;
            }
        ```

        - 특징
            - 생성자 호출 시점에 딱 1번만 호출되는 것이 보장된다.
            - **불변, 필수** 의존 관계에 사용
            - **생성자가 딱 1개만 있으면 `@Autowired` 를 생략해도 자동 주입된다**. 물론 스프링 빈에만 해당한다.

            ```java
            @Component
            public class OrderServiceImpl implements OrderService{
            
                private final MemberRepository memberRepository;
                private final DiscountPolicy discountPolicy;
            
            //    @Autowired // 생략 가능
                public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
                    this.memberRepository = memberRepository;
                    this.discountPolicy = discountPolicy;
                }
            ```

    - 수정자 주입 (setter 주입)
        - = setter라 불리는 필드의 값을 변경하는 수정자 메서드를 통해서 의존관계를 주입하는 방법
        - 특징
            - 선택, 변경 가능성이 있는 의존 관계에 사용
            - 자바빈 프로퍼티 규약의 수정자 메서드 방식을 사용하는 방법
            - 자바빈 프로퍼티 규약이란?
                - setXxx, getXxx 라는 메서드를 통해서 값을 읽거나 수정하는 규칙

                ```java
                class Data {
                 private int age;
                 public void setAge(int age) {
                	 this.age = age;
                 }
                 public int getAge() {
                	 return age;
                 }
                }
                ```

            - `@Autowired` 의 기본 동작은 주입할 대상이 없으면 오류가 발생한다.
            - 주입할 대상이 없어도 동작하게 하려면
                - `@Autowired(required = false)`로 지정하면 된다
    - 필드 주입
        - = 이름 그대로 필드에 바로 주입하는 방법

        ```java
        @Component
        public class OrderServiceImpl implements OrderService {
         @Autowired
         private MemberRepository memberRepository;
         @Autowired
         private DiscountPolicy discountPolicy;
        }
        ```

        - 특징
            - 코드가 간결하다.
            - 하지만 외부에서 의존성을 주입하기 어려워 순수 Java 테스트가 힘들어진다는 치명적인 단점이 있다.
            - final 키워드를 사용할 수 없어 불변성을 보장하기 어렵다.
            - DI 프레임워크가 없으면 아무것도 할 수 없다.
            - → 사용하지 말자!
            - 두 가지 경우에만 사용 가능하긴 하다.
                - 애플리케이션의 실제 코드와 관계 없는 테스트 코드
                - 스프링 설정을 목적으로 하는 @Configuration 같은 곳에서만 특별한 용도로 사용
    - 일반 메서드 주입
        - = 일반 메서드를 통해서 주입받을 수 있다.

        ```java
        @Component
        public class OrderServiceImpl implements OrderService {
        	private MemberRepository memberRepository;
        	private DiscountPolicy discountPolicy;
        	 @Autowired
        	 public void init(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        	 this.memberRepository = memberRepository;
        	 this.discountPolicy = discountPolicy;
        	}
        }
        ```

        - 특징
            - 한 번에 여러 필드를 주입받을 수 있다.
            - 일반적으로 잘 사용하지 않는다.
- 의존관계 자동 주입은 어떠한 경우에만 동작하나요?
    - 의존관계 자동 주입은 스프링 컨테이너가 관리하는 스프링 빈이어야 동작한다.
    - 스프링 빈이란?
        - `@Component`, `@Service`, `@Repository`, `@Configuration`, `@Bean` 등으로 **컨테이너에 등록된** 객체. 컨테이너가 생명주기와 의존관계를 책임진다.
    - 스프링 빈이 아닌 Member 같은 클래스에서 `@Autowired` 코드를 적용해도 아무 기능도 동작하지 않는다.

## 옵션 처리

- 주입할 스프링 빈이 없어도 동작해야 할 때가 있다.
- 자동 주입 대상을 옵션으로 처리하는 방법은?
    - `@Autowired(required=false)`
        - 자동 주입할 대상이 없으면 수정자 메서드 자체가 호출 안 됨

        ```java
        //호출 안됨
        @Autowired(required = false)
        public void setNoBean1(Member member) {
         System.out.println("setNoBean1 = " + member);
        }
        ```

    - `org.springframework.lang.@Nullable`
        - 자동 주입할 대상이 없으면 `null`이 입력된다.

        ```java
        //null 호출
        @Autowired
        public void setNoBean2(@Nullable Member member) {
         System.out.println("setNoBean2 = " + member);
        }
        ```

    - `Optional<>`
        - 자동 주입할 대상이 없으면 `Optional.empty`가 입력된다.

        ```java
        //Optional.empty 호출
        @Autowired(required = false)
        public void setNoBean3(Optional<Member> member) {
         System.out.println("setNoBean3 = " + member);
        }
        ```


## 생성자 주입을 선택해라

- 항상 생성자 주입을 선택해라! 그리고 가끔 옵션이 필요하면 수정자 주입을 선택해라. 필드 주입은 사용하지 않는게 좋다.
- 그 이유는?
    - 불변
        - 대부분의 의존관계 주입은 한번 일어나면 애플리케이션 종료시점까지 의존관계를 변경할 일이 없다. 오히려 대부분의 의존관계는 애플리케이션 종료 전까지 변하면 안된다.(불변해야 한다.)
        - 수정자 주입을 사용하면, setXxx 메서드를 public으로 열어두어야 한다.
        - 누군가 실수로 변경할 수 도 있고, 변경하면 안되는 메서드를 열어두는 것은 좋은 설계 방법이 아니다.
        - 생성자 주입은 객체를 생성할 때 딱 1번만 호출되므로 이후에 호출되는 일이 없다. 따라서 불변하게 설계할 수 있다
    - 누락X
        - 수정자 의존 관계일 때 프레임워크 없이 순수한 자바 코드를 단위 테스트를 수행할 경우 memberRepository, discountPolicy 모두 의존관계 주입이 누락할 실수를 범하기 쉽다. 이 경우 실행은 되지만, 실행 결과 NPE(Null Point Exception)이 발생한다.
        - 하지만 생성자 주입을 사용하면 다음처럼 주입 데이터를 누락 했을 때 컴파일 오류가 발생한다. 그리고 IDE에서 바로 어떤 값을 필수로 주입해야 하는지 알 수 있다.
    - final 키워드
        - 생성자 주입을 사용하면 필드에 final 키워드를 사용할 수 있다.
        - 그래서 생성자에서 혹시라도 값이 설정되지 않는 오류를 컴파일 시점에 막아준다.
        - `java: variable discountPolicy might not have been initialized`
        - 컴파일 오류는 세상에서 가장 빠르고, 좋은 오류다!
        - 수정자 주입을 포함한 나머지 주입 방식은 모두 생성자 이후에 호출되므로, 필드에 final 키워드를 사용할 수 없다. 오직 생성자 주입 방식만 final 키워드를 사용할 수 있다.

## 롬복과 최신 트렌드

- 롬복 사용 방법
    - 롬복 라이브러리가 제공하는 `@RequiredArgsConstructor` 기능을 사용하면 final이 붙은 필드를 모아서 생성자를 자동으로 만들어준다

    ```java
    @Component
    @RequiredArgsConstructor
    public class OrderServiceImpl implements OrderService{
    
        private final MemberRepository memberRepository;
        private final DiscountPolicy discountPolicy;
    ```

- 생성자 관련 최신 트렌드
    - 최근에는 생성자를 딱 1개 두고, `@Autowired` 를 생략하는 방법을 주로 사용한다.
    - 여기에 Lombok 라이브러리의 `@RequiredArgsConstructor` 함께 사용하면 기능은 다 제공하면서, 코드는 깔끔하게 사용할 수 있다.
- 롬복 라이브러리 적용 방법

    ```java
    plugins {
    	id 'java'
    	id 'org.springframework.boot' version '3.5.4'
    	id 'io.spring.dependency-management' version '1.1.7'
    }
    
    group = 'hello'
    version = '0.0.1-SNAPSHOT'
    
    java {
    	toolchain {
    		languageVersion = JavaLanguageVersion.of(21)
    	}
    }
    
    //lombok 설정 추가 시작
    configurations {
    	compileOnly {
    		extendsFrom annotationProcessor
    	}
    }
    // lombok 설정 추가 끝
    
    repositories {
    	mavenCentral()
    }
    
    dependencies {
    	implementation 'org.springframework.boot:spring-boot-starter'
    
    	//lombok 라이브러리 추가 시작
    	compileOnly 'org.projectlombok:lombok'
    	annotationProcessor 'org.projectlombok:lombok'
    	testCompileOnly 'org.projectlombok:lombok'
    	testAnnotationProcessor 'org.projectlombok:lombok'
    	//lombok 라이브러리 추가 끝
    
    	testImplementation('org.springframework.boot:spring-boot-starter-test') {
    		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    	}
    //	testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
    }
    
    tasks.named('test') {
    	useJUnitPlatform()
    }
    ```


## 조회 빈이 2개 이상 - 문제 발생

- 같은 타입이 2개 이상이면 발생하는 문제는?
    - 모호성 예외(`NoUniqueBeanDefinitionException`)가 발생한다.
    - `expected single matching bean but found 2: fixDiscountPolicy, rateDiscountPolicy`
- 위 문제를 하위 타입으로 해결하면 안 되는 이유는?
    - **DIP(의존성 역전 원칙) 위배**: 구현체에 직접 의존하게 됩니다.
    - 이름만 다르고, 완전히 똑같은 타입의 스프링 빈이 2개 있을 때 **여전히 충돌**이 생길 수 있다.
    - 테스트나 교체가 어려워져 **유연성**이 떨어집니다.

## @Autowired 필드 명, @Qualifier, @Primary

- 조회 대상 빈이 2개 이상일 때 해결 방법
- @Autowired 필드 명 매칭
    - **먼저 타입 매칭을 시도하고, 이때 여러 빈이 있으면 필드 이름, 파라미터 이름으로 빈 이름을 추가 매칭한다.**
    - 기존 코드

    ```java
    @Autowired
    private DiscountPolicy discountPolicy
    ```

    - 필드 명을 빈 이름으로 변경

    ```java
    @Autowired
    private DiscountPolicy rateDiscountPolicy
    ```

    - 필드 명이 rateDiscountPolicy 이므로 정상 주입된다.
- @Qualifier 사용
    - 추가 구분자를 붙여주는 방법
    - 주입시 추가적인 방법을 제공하는 것이지 빈 이름을 변경하는 것
      은 아니다.
    - @Qualifier가 적용되는 순서
        1. @Qualifier끼리 매칭
        2. 빈 이름 매칭
        3. NoSuchBeanDefinitionException 예외 발생
    - 빈 등록 방법
        - 빈 등록시 @Qualifier를 붙여 준다.

        ```java
        @Component
        @Qualifier("mainDiscountPolicy")
        public class RateDiscountPolicy implements DiscountPolicy {}
        ```

        ```java
        @Component
        @Qualifier("fixDiscountPolicy")
        public class FixDiscountPolicy implements DiscountPolicy {}
        ```

        - 직접 빈 등록시에도 @Qualifier를 동일하게 사용할 수 있다.

        ```java
        @Bean
        @Qualifier("mainDiscountPolicy")
        public DiscountPolicy discountPolicy() {
         return new ...
        }
        ```

    - 주입 방법
        - 주입시에 @Qualifier를 붙여주고 등록한 이름을 적어준다.
        - 생성자 자동 주입 예시

        ```java
        @Autowired
        public OrderServiceImpl(MemberRepository memberRepositoy, @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
         this.memberRepository = memberRepository;
         this.discountPolicy = discountPolicy;
        }
        ```

        - 수정자 자동 주입 예시

        ```java
        @Autowired
        public DiscountPolicy setDiscountPolicy(@Qualifier("mainDiscountPolicy")
        DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
        }
        ```

    - @Qualifier 로 주입할 때 @Qualifier("mainDiscountPolicy") 를 못 찾으면 어떻게 될까?
        - mainDiscountPolicy라는 이름의 스프링 빈을 추가로 찾는다.
        - 하지만 경험상 @Qualifier 는 @Qualifier 를 찾는 용도로만 사용하는게 명확하고 좋다
- @Primary 사용
    - 우선순위를 정하는 방법
    - @Autowired 시에 여러 빈이 매칭되면 @Primary 가 우선권을 가진다.
    - 예시
        - rateDiscountPolicy 가 우선권을 가지도록 하자.

        ```java
        @Component
        @Primary
        public class RateDiscountPolicy implements DiscountPolicy {}
        @Component
        public class FixDiscountPolicy implements DiscountPolicy {}
        ```

        - 사용 코드

        ```java
        //생성자
        @Autowired
        public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
         this.memberRepository = memberRepository;
         this.discountPolicy = discountPolicy;
        }
        //수정자
        @Autowired
        public DiscountPolicy setDiscountPolicy(DiscountPolicy discountPolicy) {
         this.discountPolicy = discountPolicy;
        }
        ```

        - 코드를 실행해보면 문제 없이 @Primary 가 잘 동작하는 것을 확인할 수 있다.
- 방법들 비교
- @Qualifier 단점 vs @Primary 장점
    - @Qualifier는 모든 코드에 @Qualifier 를 붙여주어야 한다.
    - @Primary 를 사용하면 이렇게 @Qualifier 를 붙일 필요가 없다.
- @Qualifier, @Primary 활용
    - 코드에서 자주 사용하는 메인 데이터베이스의 커넥션을 획득하는 스프링 빈이 있고, 코드에서 특별한 기능으로 가끔 사용하는 서브 데이터베이스의 커넥션을 획득하는 스프링 빈이 있다고 생각해보자.
    - **메인 데이터베이스의 커넥션을 획득하는 스프링 빈은 @Primary 를 적용해서 조회하는 곳에서 @Qualifier 지정 없이 편리하게 조회하고,**
    - **서브 데이터베이스 커넥션 빈을 획득할 때는 @Qualifier 를 지정해서 명시적으로 획득 하는 방식으로 사용하면 코드를 깔끔하게 유지할 수 있다.**
    - 물론 이때 메인 데이터베이스의 스프링 빈을 등록할 때 @Qualifier 를 지정해주는 것은 상관없다.
- @Qualifier vs @Primary 우선 순위
    - @Primary 는 기본값 처럼 동작하는 것이고, @Qualifier 는 매우 상세하게 동작한다.
    - 스프링은 자동보다는 수동이, 넒은 범위의 선택권 보다는 좁은 범위의 선택권이 우선 순위가 높다.
    - 따라서 여기서도 @Qualifier 가 우선권이 높다.

## 애노테이션 직접 만들기

- `@Qualifier("mainDiscountPolicy")`  단점 및 해결 방안
    - @Qualifier("mainDiscountPolicy") 이렇게 문자를 적으면 컴파일 시 타입 체크가 안된다.
    - 애노테이션을 만들어서 문제를 해결할 수 있다.
- 애노테이션 만드는 방법
    - `public @interface MainDiscountPolicy`

        ```java
        package hello.core.annotation;
        
        import org.springframework.beans.factory.annotation.Qualifier;
        import java.lang.annotation.*;
        
        @Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @Qualifier("mainDiscountPolicy")
        public @interface MainDiscountPolicy {
        }
        
        ```

    - `public class RateDiscountPolicy implements DiscountPolicy`

        ```java
        package hello.core.discount;
        
        import hello.core.annotation.MainDiscountPolicy;
        import hello.core.member.Grade;
        import hello.core.member.Member;
        import org.springframework.stereotype.Component;
        
        @Component
        @MainDiscountPolicy // 이렇게 적용
        public class RateDiscountPolicy implements DiscountPolicy{
        
            private int discountRate = 10; // 할인율 10%
        
            @Override
            public int discount(Member member, int price) {
                if (member.getGrade() == Grade.VIP) {
                    return price * discountRate / 100; // VIP 회원에게 할인 적용
                } else {
                    return 0;
                }
            }
        }
        
        ```

    - 생성자 자동 주입 : `public OrderServiceImpl`

        ```java
            @Autowired
            public OrderServiceImpl(MemberRepository memberRepository, @MainDiscountPolicy DiscountPolicy discountPolicy) {
                this.memberRepository = memberRepository;
                this.discountPolicy = discountPolicy;
            }
        ```

    - 수정자 자동 주입 : `public void setDiscountPolicy`

        ```java
            @Autowired
            public void setDiscountPolicy(@MainDiscountPolicy DiscountPolicy discountPolicy) {
                this.discountPolicy = discountPolicy;
            }
        ```

- 애노테이션의 장점
    - 애노테이션에는 상속이라는 개념이 없다.
    - 이렇게 여러 애노테이션을 모아서 사용하는 기능은 스프링이 지원해주는 기능이다.
    - @Qualifier 뿐만 아니라 다른 애노테이션들도 함께 조합해서 사용할 수 있다.
    - 단적으로 @Autowired 도 재정의 할 수 있다.
    - 물론 스프링이 제공하는 기능을 뚜렷한 목적 없이 무분별하게 재정의 하는 것은 유지보수에 더 혼란만 가중할 수 있다.

## 조회한 빈이 모두 필요할 때, List, Map

- 의도적으로 정말 해당 타입의 스프링 빈이 다 필요한 경우도 있다.
- Spring 컨테이너는 특정 타입의 모든 Bean들을 List나 Map 형태로 주입하는 기능을 지원한다.
- 이를 통해 여러 구현체 중 하나를 동적으로 선택하는 전략 패턴 등을 쉽게 구현할 수 있다.
- 코드
    - 예를 들어서 할인 서비스를 제공하는데, 클라이언트가 할인의 종류(rate, fix)를 선택할 수 있다고 가정해보자.

    ```java
    package hello.core.autowired;
    
    import hello.core.AutoAppConfig;
    import hello.core.discount.DiscountPolicy;
    import hello.core.member.Grade;
    import hello.core.member.Member;
    import org.junit.jupiter.api.Test;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.annotation.AnnotationConfigApplicationContext;
    
    import java.util.List;
    import java.util.Map;
    
    import static org.assertj.core.api.Assertions.assertThat;
    
    public class AllBeanTest {
        @Test
        void findAllBean() {
            ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);
            DiscountService discountService = ac.getBean(DiscountService.class);
    
            Member member = new Member(1L, "userA", Grade.VIP);
            int discountPrice = discountService.discount(member, 10000, "fixDiscountPolicy");
            assertThat(discountService).isInstanceOf(DiscountService.class);
            assertThat(discountPrice).isEqualTo(1000);
        }
    
        static class DiscountService {
            private final Map<String, DiscountPolicy> policyMap;
            private final List<DiscountPolicy> policies;
    
            public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policies) {
                this.policyMap = policyMap;
                this.policies = policies;
                System.out.println("policyMap = " + policyMap);
                System.out.println("policies = " + policies);
            }
    
            public int discount(Member member, int price, String discountCode) {
                DiscountPolicy discountPolicy = policyMap.get(discountCode);
                System.out.println("discountCode = " + discountCode);
                System.out.println("discountPolicy = " + discountPolicy);
                return discountPolicy.discount(member, price);
            }
        }
    }
    ```

- 위의 코드를 설명해 보시오.
    - 로직 분석
        - DiscountService
            - Map으로 모든 `DiscountPolicy` 를 주입받는다. 이때 `fixDiscountPolicy`, `rateDiscountPolicy` 가 주입된다.
        - `discount()` 메서드
            - discountCode로 "fixDiscountPolicy"가 넘어오면 map에서 `fixDiscountPolicy` 스프링 빈을 찾아서 실행한다.
            - 물론 “rateDiscountPolicy”가 넘어오면 `rateDiscountPolicy` 스프링 빈을 찾아서 실행한다.
    - 주입 분석
        - `Map<String, DiscountPolicy>`
            - map의 키에 스프링 빈의 이름을 넣어주고, 그 값으로 DiscountPolicy 타입으로 조회한 모든 스프링 빈을 담아준다.
        - `List<DiscountPolicy>`
            - DiscountPolicy 타입으로 조회한 모든 스프링 빈을 담아준다.
        - 만약 해당하는 타입의 스프링 빈이 없으면
            - 빈 컬렉션이나 Map을 주입한다.
    - 스프링 컨테이너를 생성하면서 스프링 빈 등록하는 방법
        - 스프링 컨테이너는 생성자에 클래스 정보를 받는다.
        - 여기에 클래스 정보를 넘기면 해당 클래스가 스프링 빈으로 자동 등록된다.
        - `ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);`
            - `new AnnotationConfigApplicationContext()`를 통해 스프링 컨테이너를 생성한다.
            - `AutoAppConfig.class` , `DiscountService.class`를 파라미터로 넘기면서 해당 클래스를 자동으로 스프링 빈으로 등록한다.

## 자동, 수동의 올바른 실무 운영 기준

- **편리한 자동 기능을 기본**으로 사용하자. 그 이유는?
    - 스프링은 `@Component` 뿐만 아니라 `@Controller` , `@Service` , `@Repository` 처럼 계층에 맞추어 일반적인 애플리케이션 로직을 자동으로 스캔할 수 있도록 지원한다.
    - 거기에 더해서 최근 스프링 부트는 컴포넌트 스캔을 기본으로 사용하고, 스프링 부트의 다양한 스프링 빈들도 조건이 맞으면 자동으로 등록하도록 설계했다.
    - 결정적으로 자동 빈 등록을 사용해도 OCP, DIP를 지킬 수 있다.
- 수동 빈 등록의 장점은?
    - 설정 정보를 기반으로 애플리케이션을 구성하는 부분과 실제 동작하는 부분을 명확하게 나눌 수 있다.
- 수동 빈 등록의 단점은?
    - 스프링 빈을 하나 등록할 때 `@Component` 만 넣어주면 끝나는 일을 `@Configuration` 설정 정보에 가서 `@Bean` 을 적고, 객체를 생성하고, 주입할 대상을 일일이 적어주는 과정은 상당히 번거롭다.
    - 또 관리할 빈이 많아서 설정 정보가 커지면 설정 정보를 관리하는 것 자체가 부담이 된다.
- 애플리케이션은 크게 두 로직으로 나눌 수 있다. 두 로직은?
    - 업무 로직
        - 웹을 지원하는 컨트롤러, 핵심 비즈니스 로직이 있는 서비스, 데이터 계층의 로직을 처리하는 리포지토리등이 모두 업무 로직이다.
        - 보통 비즈니스 요구사항을 개발할 때 추가되거나 변경된다
    - 기술 지원 로직
        - 기술적인 문제나 공통 관심사(AOP)를 처리할 때 주로 사용된다.
        - 데이터베이스 연결이나, 공통 로그 처리처럼 업무 로직을 지원하기 위한 하부 기술이나 공통 기술들이다.
- 각 로직은 둘 중 무엇을 사용해야 하는가? 그리고 그 이유는?
    - 업무 로직의 경우 자동 기능을 적극 사용하는 것이 좋다.
        - 업무 로직은 숫자도 매우 많고, 한번 개발해야 하면 컨트롤러, 서비스, 리포지토리 처럼 어느정도 유사한 패턴이 있다.
        - 보통 문제가 발생해도 어떤 곳에서 문제가 발생했는지 명확하게 파악하기 쉽다.
    - 기술 지원 로직의 경우 수동 기능을 사용하는 것이 좋다.
        - 기술 지원 로직은 업무 로직과 비교해서 그 수가 매우 적고, 보통 애플리케이션 전반에 걸쳐서 광범위하게 영향을 미친다.
        - 그리고 업무 로직은 문제가 발생했을 때 어디가 문제인지 명확하게 잘 드러나지만, 기술 지원 로직은 적용이 잘 되고 있는지 아닌지 조차 파악하기 어려운 경우가 많다.
        - 그래서 애플리케이션에 광범위하게 영향을 미치는 기술 지원 객체는 수동 빈으로 등록해서 딱! 설정 정보에 바로 나타나게 하는 것이 유지보수 하기 좋다.
- 비즈니스 로직 중에서 다형성을 적극 활용할 때는 둘 중 무엇을 사용해야 하는가?
    - 비즈니스 로직 중에서 다형성을 적극 활용할 때는 어떤 빈들이 주입될 지, 각 빈들의 이름은 무엇일지 코드만 보고 한번에 쉽게 파악하기 어렵다.
    - 이런 경우 수동 빈으로 등록하거나 또는 자동으로 하려면 특정 패키지에 같이 묶어두는게 좋다!
    - 수동 빈 등록 예시

        ```java
        @Configuration
        public class DiscountPolicyConfig {
        
         @Bean
         public DiscountPolicy rateDiscountPolicy() {
        	 return new RateDiscountPolicy();
         }
         @Bean
         public DiscountPolicy fixDiscountPolicy() {
        	 return new FixDiscountPolicy();
         }
        }
        ```

        - 이 설정 정보만 봐도 한눈에 빈의 이름은 물론이고, 어떤 빈들이 주입될지 파악할 수 있다.
- 스프링과 스프링 부트가 자동으로 등록하는 수 많은 빈들은 어떻게 해야 하는가?
    - 스프링 자체를 잘 이해하고 스프링의 의도대로 잘 사용하는게 중요하기에 자동 기능 그대로 사용해야 한다.
    - 스프링 부트의 경우 DataSource 같은 데이터베이스 연결에 사용하는 기술 지원 로직까지 내부에서 자동으로 등록하는데, 이런 부분은 메뉴얼을 잘 참고해서 스프링 부트가 의도한 대로 편리하게 사용하면 된다.
- 스프링 부트가 아니라 내가 직접 기술 지원 객체를 스프링 빈으로 등록한다면?
    - 수동으로 등록해서 명확하게 드러내는 것이 좋다.
- 정리
    - **자동 기능**
        - **편리한 자동 기능을 기본으로 사용하자**
        - **업무 로직**
        - **스프링과 스프링 부트가 자동으로 등록하는 수 많은 빈들**
    - **수동 등록**
        - **기술 지원 로직**
        - **다형성을 적극 활용하는 비즈니스 로직**
        - **직접 등록하는 기술 지원 객체**