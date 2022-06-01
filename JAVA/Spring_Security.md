## 이 글을 쓰게 된 이유
처음으로 spring security를 사용하게 되면서 큰 어려움에 부딪혔다.. 그리고 막상 찾아보려고 하니 마땅한 자료가 조금 부족한 느낌이었고 공식 문서를 보면서 학습하려니 쉽지 않았다. 아무튼 나 같은 사람들을 위해 내가 며칠동안 고민하고 이해한 내용들을 정리해보려고 한다. 물론 spring security은 더 깊고 넓은 내용이 있다. 일단 큰 틀과 그 틀을 활용하는 기본적인 방법들에 대해서 찍먹해보자.

## 0. Spring Security 학습 방향
spring security를 학습하며 '다른 부분을 모르고 지금 이걸 읽어도 되나?'라는 생각이 지속적으로 들었다. 나는 야생형 학습 스타일이라서 사용하면서 부족한 부분을 채우는 편인데도 불구하고 spring security는 부족한 부분이 너무 많아서 밑 빠진 독에 물을 붓는 느낌이 강했다. 
spring security는 엄청나게 많은 기능을 편리하게 사용할 수 있게 해주는 툴이다. 그래서 사용에 초점을 두되 돌아는 과정을 이해해서 이 과정에서 내가 어떤 부분을 구현해야하는지에 초점을 맞춰서 학습을 하려고 한다. 우선 순위를 두자면 다음과 같다.

1. spring security가 돌아가는 과정을 이해한다(공식문서 기반). 이번 글

2. 이 과정에서 사용자가 필수로 구현할 부분을 추출해서 이해한다. coming soon

3. 이 과정에서 사용자가 응용하고 싶은 부분을 추출해서 이해한다. coming soon


## 1. Spring Security란?
>Spring Security is a framework that provides authentication, authorization, and protection against common attacks. With first class support for securing both imperative and reactive applications, it is the de-facto standard for securing Spring-based applications. -Spring Security docs-

위는 스프링 공식 문서에서 spring security에 대해 간단하게 설명한 것이다. 더 간단하게 얘기하면 spring security는 spring을 활용한 애플리케이션에서 인증과 인가를 담당하고, 일반적인(알려진) 해킹 공격에 대해 보호해주는 사실상의 보안 표준이라는 말이다. 
여기서 우리는 spring security가 제공하는 기능을 크게 2가지로 나눌 수 있다.

1. 인증과 인가
2. 일반적인 해킹 공격으로부터 보호(CSRF, 세션고정 등)

## 2. Spring Security 동작 과정
![](https://velog.velcdn.com/images/younghoondoodoom/post/a7542545-12bc-432c-964f-bd2e3be12584/image.png)
출처 : https://docs.spring.io/spring-security/reference/servlet/architecture.html

spring security는 요청이 들어오면 Servlet FilterChain을 자동으로 구성하고 거치게한다. FilterChain은 보다시피 여러 filter를 chain형태로 묶어놓은 것을 얘기한다. 그렇다면 Filter가 무엇이고 왜 spring security는 filter를 이용할까?

지금부터 filter와 관련해서 어떻게 spring security가 동작하는지 순차적으로 지식을 쌓아가며 알아보자!(물론 간단하게만)

### 2.1 Filter
![](https://velog.velcdn.com/images/younghoondoodoom/post/46527fd9-d730-401e-bb8d-c084614e2b28/image.png)
**Filter는 J2EE 표준 스펙 기능으로 디스패처 서블릿에 요청이 전달되기 전후에 url 패턴에 맞는 모든 요청에 대해 필터링을 할 수 있는 기능을 제공한다.**
즉, 스프링 컨테이너에서 제공되는 기능이 아니라 톰캣과 같은 웹 컨테이너에 의해 관리가 된다.
Filter는 전역적으로 동작해야하는 보안 검사(CSRF, XSS 방어 등)를 하여 올바른 요청이 아닐 경우 차단을 한다. 그러면 요청이 스프링 컨테이너까지 전달되지 못해 안정성을 더욱 높일 수 있다. 이러한 강력한 기능 때문에 spring security는 filter를 이용하는 것이다. 

과거에는 Filter는 서블릿 기술이라서 spring 빈으로 등록할 수 없었다. (오래된 글이나 책을 봐도 filter는 빈으로 관리할 수 없다고 나온다.) 그렇다면 우리는 어떻게 필터를 활용할 수 있을까?

### 2.2 DelegatingFilterProxy
![](https://velog.velcdn.com/images/younghoondoodoom/post/df58cd40-6cd3-415a-a94e-70b937b02156/image.png)
filter는 서블릿 기술이어서 스프링 컨테이너 밖에서 동작한다. 그래서 과거에는 스프링으로 filter를 통제할 수 없었다. 하지만 DelegatingFilterProxy라는 기술으로 spring bean으로 filter를 등록하여 사용할 수 있게 되었다. 
요약하면 DelegatingFilterProxy를 활용하여 bean을 등록을 할 수 있구나 하면 된다.

### 2.3 FilterChainProxy
![](https://velog.velcdn.com/images/younghoondoodoom/post/f6fd1244-f645-4cf1-8b90-2215d22b9ff7/image.png)
그리고 Spring Security는 DelegatingFilterProxy에 의해 빈으로 등록된 FilterChainProxy를 활용해 SecurityFilterChain에 있는 filter들을 사용할 수 있다. 즉, proxy를 2번 활용하여 SecurityFilterChain 안에 있는 filter들을 활용할 수 있게 하는 것이다.

정리)
DelegatingFilterProxy -> FilterChainProxy -> SecurityFilterChain(여기에 spring 필터들이 있음)

### 2.4 SecurityFilterChain
![](https://velog.velcdn.com/images/younghoondoodoom/post/bd6d5037-d9aa-4e41-935b-08d7e9d78b70/image.png)
SecurityFilterChain은 FilterChainProxy에서의 요청에 대해 호출해야하는 스프링 보안 필터를 결정하는데에 쓰인다.

### 2.5 FilterChainProxy를 굳이 사용하는 이유
여기서 한가지 의문이 들 수 있다. DelegatingFilterProxy로 bean을 filter로 등록할 수 있다고 했으면서 왜 굳이 FilterChainProxy를 사용해 프록시를 한번 더 사용하여 등록을 할까? FilterChainProxy를 사용하면 얻을 수 있는 장점이 있기 때문이다.

1. **모든 Spring Security의 서블렛 이용에 대한 시작점을 제공한다.** 덕분에 문제가 생기면 FilterChainProxy에 디버깅 포인트를 잡아서 빠르게 오류를 수정할 수 있다.
2. **Spring Security의 중심점으로 잡음으로서 선택이 아닌 필수들인 작업들을 누락없이 실행할 수 있다.** 예를 들어 메모리 낭비를 방지해 SecurityContext를 지우거나 Http Firewall를 적용하여 특정 공격으로부터 어플리케이션을 보호한다.
3. **SecurityFilterChain의 호출 시키를 유연하게 조절할 수 있다.** 원래 서블렛 컨테이너는 URL을 따라서만 호출할지 안할지를 결정할 수 있다. 하지만 FilterChainProxy를 사용하면 RequestMatcher 인터페이스를 활용하여 HttpServletRequest의 조건을 걸어 호출이 가능하다.

4. **FilterChainProxy는 어떤 SecurityFilterChain를 사용할지 결정하는데에 쓰일 수도 있다.** 즉, 한 어플리케이션 안에서 여러가지 인증 방식(session, jwt등)을 사용하는데에 설정을 완전히 분리할 수 있는 환경을 만들어줌.
![](https://velog.velcdn.com/images/younghoondoodoom/post/ea0b9822-e099-444c-ac3c-26dd621515e8/image.png)

### 2.6 Security Filters 순서
공식 문서를 보면 SecurityFilterChain 안에 있는 Filter의 순서는 중요하지만 일반적으로 우리가 그 순서에 대해서 알고 있어야 할 필요는 없다고 설명한다. 하지만 가끔 그 순서를 알고 있으면 좋을 수도 있다고 한다.
궁금하면 [여기](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-security-filters)에서 확인하면 된다.

## Spring Security 예외 발생 과정
Spring Security는 AccessDeniedException과 AuthenticationException을 http response로 바꿔주기 위해 ExceptionTranslationFilter라는 필터를 활용한다.
물론 ExceptionTranslationFilter는 SecurityFilterChain에 들어있는 필터중 하나다.

이 과정은 다음과 같은 과정으로 일어난다.
![](https://velog.velcdn.com/images/younghoondoodoom/post/0e158e13-637e-4a71-91eb-3f799e649404/image.png)

1. 일단, ExceptionTranslationFilter는 FilterChain.doFilter(request, response)를 호출하여 다음 filter로 바로 넘어가게 한다.
2. 만약 유저가 인증이 안되었거나 AuthenticationException을 발생 시키면 Authentication(인증)을 시작한다.(Authentication에 대한 자세한건 다음 편에)
3. 인증이 실패하여 AccessDeniedException이 발생하면 AccessDeniedHandler를 호출한다.

이를 코드로 이해하면 다음과 같다.
```java
try {
	filterChain.doFilter(request, response);
} catch (AccessDeniedException | AuthenticationException ex) {
	if (!authenticated || ex instanceof AuthenticationException) {
		startAuthentication();
	} else {
		accessDenied();
	}
}
```
위에서 설명한거와 같다. 다음 필터로 넘어가도록 하는데 예외가 발생하면 인증을 시작하고 인증에 실패하면 예외를 핸들링하는 로직이 실행된다.

즉, 예외가 발생하지 않으면 ExceptionTranslationFilter는 그냥 넘어가게 된다.

## 정리 
spring security가 어떻게 동작하는지, 그리고 예외처리는 어떤 식으로 진행되는지 보았다. 다음 편에서는 구체적으로 어떻게 인증과 인가가 진행이되는지 그리고 개발자는 어떤 부분을 설정하고 구현해야하는지 살펴보도록 하자.

만약 틀리거나 부족한 부분이 있다면 댓글을 남겨주세요!
