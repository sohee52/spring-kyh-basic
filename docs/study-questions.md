# 기본편 질문

- SOLID : 좋은 객체 지향 설계의 5가지 원칙과 의미를 말하시오.

- 아래 코드의 문제점과 해결 방법을 말하시오.

```java
package hello.core.member;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class MemberServiceImpl implements MemberService{

    private final MemberRepository memberRepository = new MemoryMemberRepository();

    @Override
    public void join(Member member) {
        memberRepository.save(member);
    }

    @Override
    public Member findMember(Long memberId) {
        return memberRepository.findById(memberId);
    }
}

```

```java
package hello.core;

import hello.core.member.Grade;
import hello.core.member.Member;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class MemberApp {
    public static void main(String[] args) {

        MemberService memberService = new memberServiceImpl();
        
        Member member = new Member(1L, "memberA", Grade.VIP);
        memberService.join(member);

        Member findMember = memberService.findMember(1L);
        System.out.println("new member = " + member.getName());
        System.out.println("find Member = " + findMember.getName());
    }
}
```

- 제어의 역전 IoC (Inversion of Control)과 의존 관계 주입 DI (Dependency Injection), DI 컨테이너에 대해서 설명하시오.

- 스프링 컨테이너의 의미와 생성 과정을 말하시오.

- 빈을 조회하는 기본적인 방법 3가지를 말하시오.
- `MemberRepository bean = ac.getBean(MemberRepository.class);`와 같이 타입으로 조회하였을 때 같은 타입이 둘 이상 있으면 중복 오류가 발생한다. 어떻게 해결해야 하는가?
- `DiscountPolicy bean = ac.getBean(DiscountPolicy.class);`부모 타입으로 조회시, 자식이 둘 이상 있으면, 중복 오류가 발생한다. 어떻게 해결해야 하는가?
- 특정 타입을 모두 조회하고 싶을 때 사용하는 함수는?

- 스프링 없는 순수한 DI 컨테이너의 문제점을 말하시오.
- 싱글톤 패턴이란?
- 싱글톤 패턴 직접 구현하는 방법은?
- 싱글톤 패턴의 단점
- 스프링 컨테이너 (싱글톤 컨테이너) 장점
- 싱글톤 레지스트리란?
- 싱글톤 컨테이너 사용 방법
- 싱글톤 방식을 사용할 때 주의해야할 점은?
- 아래 코드의 문제점과 해결 방안을 말하시오.

```java
package hello.core.singleton;

public class StatefulService {
    private int price;
    
    public void order(String name, int price) {
        System.out.println("name = " + name + " price = " + price);
        this.price = price;
    }

    public int getPrice() {
        return price;
    }
}
```

```java
package hello.core.singleton;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;

public class StatefulServiceTest {
    @Test
    void statefulServiceSingleton() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
        StatefulService statefulService1 = ac.getBean("statefulService", StatefulService.class);
        StatefulService statefulService2 = ac.getBean("statefulService", StatefulService.class);

        statefulService1.order("userA", 10000);
        statefulService2.order("userB", 20000);

        int price = statefulService1.getPrice();

        System.out.println("price = " + price);
        Assertions.assertThat(statefulService1.getPrice()).isEqualTo(20000);
    }

    static class TestConfig {
        @Bean
        public StatefulService statefulService() {
            return new StatefulService();
        }
    }
}
```

- 아래 코드는 싱글톤 패턴인가? 이유와 함께 설명하시오.

```java
package hello.core;

@Configuration
public class AppConfig {
    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemoryMemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(
                memberRepository(), discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }
}

```

- 컴포넌트 스캔이란?
- 컴포넌트 스캔과 자동 의존관계 주입 동작 방식
- 탐색할 패키지의 시작 위치 지정 **방법**
- 탐색할 패키지의 시작 위치 지정 **관례**
- 컴포넌트 스캔 기본 대상
- `includeFilters` 와 `excludeFilters`  사용 방법
- FilterType 옵션 - `ANNOTATION` , `ASSIGNABLE_TYPE`
- 자동 빈 등록 vs 자동 빈 등록
- 스프링에서 수동 빈 등록 vs 자동 빈 등록
- 스프링 부트에서 수동 빈 등록 vs 자동 빈 등록

- 의존관계 주입 4가지 방법과 특징을 말하시오. 이 중 어떤 방법이 권장되는지 이유와 함께 말하시오.
- 자동 주입 대상을 옵션으로 처리하는 방법 3가지 말하시오.
- `@RequiredArgsConstructor` 에 대해서 설명하시오.
- 조회 대상 빈이 2개 이상일 때 해결 방법 3가지 말하시오.
- @Qualifier, @Primary를 어떻게 활용해야 하는지, 이 둘 중 어느 것이 더 우선 순위가 높은지 말하시오.
- @Qualifier의 단점과 해결 방법을 말하시오
- 해당 타입의 스프링 빈이 다 필요한 경우 모든 Bean들을 어떤 형태로 주입하여야 하는가?
- 자동, 수동의 올바른 실무 운영 기준을 말하시오.

- 스프링 빈의 이벤트 라이프사이클은?
- 객체의 생성과 초기화를 분리하는 이유는?
- 스프링이 지원하는 빈 생명주기 콜백 방법 3가지와 어떤 걸 선택해야 하는지?

- 스코프의 종류 3가지와 의미를 말하시오.
- 빈 스코프 지정 방법을 2가지 말하시오.
- 싱글톤 스코프의 빈을 스프링 컨테이너에 요청했을 때와 프로토타입 스코프의 빈을 스프링 컨테이너에 요청했을 때의 차이를 말하시오.
- 스프링은 일반적으로 싱글톤 빈을 사용하므로, 싱글톤 빈이 프로토타입 빈을 사용하게 된다. 이때 생기는 문제점은?
- 위의 문제를 해결하는 방법을 3가지를 말하고, 실무에선 주로 어떤 방법을 사용하는지 말하시오.
- Request 스코프란?
- Request 스코프 빈은 실제 고객의 요청이 와야 생성이 되기에 애플리케이션 실행 시점에 오류가 발생할 수 있다. 이를 해결하는 방안 2가지 말하시오.
- 같은 HTTP 요청 안에서 컨트롤러와 서비스가 `getObject()`를 각각 호출하면 서로 다른 인스턴스를 받나요?