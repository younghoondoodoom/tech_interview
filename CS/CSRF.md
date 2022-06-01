## 0. 이 글을 쓰게 된 이유
JAVA SPRING MVC를 이용해서 개인 프로젝트를 하던 중에 CSRF 또 만났다. 이전에도 봤던 놈이고 개념을 안다고 생각해서 나름대로 해결을 해서 진행을 하려고 했는데 문득 관연 내가 CSRF에 대해서 완벽하게 알고있나 의문이 들었다. 그리고 지금 CSRF를 피하려고 사용하는 간편한 방법이 정말 보안 상 옳은 것인가에 대한 의문 또한 들었다. 이렇게 넘어갔다가는 나중에 호되게 당할 것 같아서 이 참에 확실하게 집고 넘어가려고 이 글을 작성하게 되었다.

## 1. CSRF(Cross-Site Request Forgery)가 뭔데
>웹 애플리케이션 취약점 중 하나로 사용자가 자신의 의지와 무관하게 공격자가 의도한 행동을 해서 특정 웹페이지를 보안에 취약하게 한다거나 수정, 삭제 등의 작업을 하게 만드는 공격 방법이다. -나무위키

사용자가 의도하진 않았지만, 자신도 모르게 가지고 있는 쿠키 등 데이터를 이용해서 서버를 공격하는 해위를 얘기한다. 어떻게 이게 가능한지 알아보자!

### 1.1 쿠키와 세션
MVC 패턴을 사용하면 사용자의 권한을 구현할 때 쿠키와 세션이라는 것이 쓰이게 된다. 이 부분을 알아야 CSRF를 이해할 수 있기 때문에 간단하게 알아보고 가자. 다음은 쿠키와 세션을 이용한 로그인을 순서대로 정리한 그림이다.

![](https://velog.velcdn.com/images/younghoondoodoom/post/ee49e2a2-08cd-42f8-9c0e-96dd8de700fe/image.png)

1. 서버는 로그인 요청이 들어오면 DB 조희를 통해 권한을 확인하고 사용자 정보를 Session에 저장하고, 이를 찾을 수 있는 키 값인 SessionId를 생성한다. (보통 Session은 메모리에 저장)
2. 서버는 저장된 세션 정보를 클라이언트(브라우저)가 사용할 수 있도록 sessionID를 Set-Cookie 헤더에 담아서 전달한다.
3. 클라이언트(브라우저)는 전달된 sessionID를 쿠키에 저장한다.
4. 클라이언트(브라우저)는 해당 도메인을 가진 서버로 요청 시 쿠키에 저장된 sessionID를 자동으로 전달한다.
5. 서버는 쿠키에 담긴 sessionID를 통해 인증된 사용자인지 여부를 확인하고 요청을 처리한다.

쿠키는 stateless한 서버를 stateful하게 사용하려고 만든 장치이고 이는 클라이언트(브라우저)에서 관리하기 때문에 보안이 취약하고 중간에 가로채기 쉽다. 만약 위와 같은 보안 인증만 있다면 해커가 쿠키를 서버의 세션 정보를 쉽게 활용할 수 있을 것이다.

## 2. CSRF 공격 과정
이제 해커가 어떤 식으로 쿠키를 이용하여 서버의 세션 정보를 취득하는지 알아보자.

위에서 대충 설명은 했지만 CSRF 공격은 다음과 같은 조건을 전제로 한다.
- 서버는 쿠키를 이용해 세션 정보에 접근할 수 있어야 한다.
- 사용자가 로그인을 하여 이미 인증을 받아 브라우저에 쿠키가 있는 상태이어야한다.
- 공격자는 서버를 공격하기 위한 요청 방법에 대해 미리 파악하고 있어야한다. 만약 요청 방법이나 예상치 못한 파라미터가 있다면 공격이 어렵다. (그 정도도 모르면 공격 안할듯)

위와 같은 환경이 조성되면 다음과 같은 과정으로 CSRF 공격이 진행된다.

![](https://velog.velcdn.com/images/younghoondoodoom/post/5a717bab-a452-4270-a3ad-31f3d4cb5e5f/image.png)

1. 사용자는 로그인을 하여 브라우저에 SessionId를 담은 쿠키가 있다.
2. 해커는 사용자에게 악성 스크립트 페이지를 클릭하도록 유도한다.
	- 메일을 통해 직접 전송되어 클릭을 유도
    - 게시물을 통해 클릭을 유도
3. 사용자가 악성 페이지에 접근 시에 악성 페이지에 있는 요청을 브라우저에 있는 쿠키와 함께 서버로 전송
4. 서버는 아무 것도 모르고 쿠키만 확인하고 요청 처리


### 2.1 CSRF 공격방식
csrf 공격은 여러 방식이 존재한다고 한다. 그 중 대표적인 GET, POST 공격에 대해 알아보자.

#### 2.1.1 GET 공격방식
```
GET http://bank.com/trasnfer?accountNumber=1122&amount=10000
```
위 예시는 사용자가 **1122라는 번호의 계좌**로 **금액 10000원**을 이체하기 위한 GET 요청이다.

```html
<a href="http://bank.com/transfer?accountNumber=5678&amount=10000">
Pictures@
</a>
```
해커는 a태그를 이용해 사용자가 클릭하도록 유도할 수 있다.


```html
<img src="http://bank.com/transfer?accountNumber=5678&amount=10000"/>
```
혹은 img 태그를 이용해 사용자가 따로 클릭 하지 않아도 페이지가 로드되면 요청은 자동적으로 실행이 된다.

GET 공격 방식은 대부분 http method를 제대로 정의 해주면 막을 수 있다.
원래의 메소드 목적에 맞게 GET은 조회를 하는 용도로만 사용해야 한다.

#### 2.1.2 POST 공격방식
```
POST http://bank.com/transfer
accountNumber=1122&amount=10000
```
아까와 마친가지인 메서드이지만 HTTP method는 POST이다.

이 경우엔 a, img 태그로는 작동하지 않는다.
따라서 해커는 다음과 같이 form태그가 필요하다.

```http
<form action="http://bank.com/transfer" method="POST">
    <input type="hidden" name="accountNumber" value="5678"/>
    <input type="hidden" name="amount" value="10000"/>
    <input type="submit" value="Pictures@"/>
</form>
```
타입이 hidden이기 때문에 사용자에게 보이지 않는다.

```html
<body onload="document.forms[0].submit()">

<form>

...
  ```
자바스크립트를 사용한다면 다음과 같이 양식을 자동으로 전송하도록 설정할 수 있다.

## 3. CSRF 방어방법
csrf를 막기 위한 방법은 여러가지가 있다고 한다. 그 중 대표적인 방법과 spring에서 추천하는 방법에 대해서 알아보자.

### 3.1 Referrer 검증
서버에서 사용자의 요청에 Referer 헤더 정보를 확인하는 방법이다. 보통이라면 호스트와 Referer 값이 일치하므로 둘을 비교한다. 대부분의 CSRF 공격은 이 검증만으로 방어가 가능하다고 한다. 
```java
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class RefererCheckInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String referer = request.getHeader("Referer"); //헤더에서 Referer 가져옴.
        String host = request.getHeader("host"); // 헤더에서 host를 가져옴
        if (referer == null || !referer.contains(host)) {
            response.sendRedirect("/");
            return false; // 같은지 체크하는 코드
        }
        return true;
    }
}
```
>Referer 요청 헤더는 현재 요청을 보낸 페이지의 절대 혹은 부분 주소를 포함합니다. 만약 링크를 타고 들어왔다면 해당 링크를 포함하고 있는 페이지의 주소가, 다른 도메인에 리소스 요청을 보내는 경우라면 해당 리소스를 사용하는 페이지의 주소가 이 헤더에 포함됩니다.Referer 헤더는 사람들이 어디로부터 와서 방문 중인지를 인식할 수 있도록 해주며 해당 데이터는 예를 들어, 분석, 로깅, 혹은 캐싱 최적화에 사용될 수도 있습니다 -MDN


Referer는 이 요청의 출처를 나타내주므로 해커의 링크를 타고 들어오면 막을 수 있다. 

### 3.2 CSRF TOKEN
이 방법은 SPRING SECURITY에서 제공하는 방법이다. 자원에 대한 변경 요청에 한해서 작동한다.
1. 서버에서 임의의 CSRF 토큰을 만들어 세션에 저장한다. 그리고 클라이언트로 그 값을 전달한다.
2. 클라이언트는 그 값을 받아 form태그 안에 input 태그를 넣어서 변경 요청을 할 때 같이 서버로 전달한다. (물론 input태그는 hidden) 
3. 서버에서 그 값이 세션에 있는 값과 동일한지 확인한다.

이를 코드로 풀면 다음과 같다
```java
// 세션에 설정
    session.setAttribute("CSRF_TOKEN", UUID.randomUUID().toString());
    
// 페이지 내 hidden 값으로 설정
    model.addAttribute("CSRF_TOKEN", session.getAttribute("CSRF_TOKEN"));
```
다음은 thymeleaf를 활용한 클라이언트 코드다.
```thymeleaf
<form th:action="@{/member/login}" method="post" class="card sign-form p-4 mt-5">
    <input type="hidden"
           th:name="${_csrf.parameterName}"
           th:value="${_csrf.token}" />
    ...
    ...
  </form>
  ```
thymeleaf에서 th:form을 이용하면 토큰을 자동으로 넣어줘서 생략도 가능하다!
다음은 csrftoken을 검증하는 intercepter 코드다
```java
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

public class CsrfTokenInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        HttpSession httpSession = request.getSession(false);
        String csrfTokenParam = request.getParameter("_csrf");
        String csrfTokenSession = (String) httpSession.getAttribute("CSRF_TOKEN");
        if (csrfTokenParam == null || !csrfTokenParam.equals(csrfTokenSession)) {
            response.sendRedirect("/");
            return false;
        }
        return true;
    }
}
```
이 방법은 서버에서 화면을 렌더링 해줄 때 csrf 토큰을 같이 건내주어 그 토큰을 가지고 있는 요청만을 받겠다는 전략이다. 


### 3.3 쿠키의 SameSite 속성
이 방법은 쿠키에 옵션을 주어서 CSRF 요청을 제한 하는 방법이다. 즉, 쿠키에 들어가는 내용물을 설정하는 방법이다.

SameSite 쿠키의 정책으로 None, Lax, Strict 세 가지 종류를 선택할 수 있고, 각각 동작하는 방식이 다르다.

- None: SameSite 가 탄생하기 전 쿠키와 동작하는 방식이 같다. None으로 설정된 쿠키의 경우 크로스 사이트 요청의 경우에도 항상 전송된다. 즉, 서드 파티 쿠키(다른 사이트에서 요청하는)도 전송돤다. 따라서, 보안적으로도 SameSite 적용을 하지 않은 쿠키와 마찬가지로 문제가 있는 방식입니다.
chrome 80부터는 SameSite를 None으로 설정하려는 경우 Secure 속성을 함께 추가해야 한다. 참고로 Secure 속성은 https 프로토콜에서만 전송이 가능하다.
- Strict: 가장 보수적인 정책. Strict로 설정된 쿠키는 크로스 사이트 요청에는 항상 전송되지 않는다. 즉, 서드 파티 쿠키는 전송되지 않고, 퍼스트 파티 쿠키만 전송.
- Lax: Strict에 비해 상대적으로 느슨한 정책. Lax로 설정된 경우, 대체로 서드 파티 쿠키는 전송되지 않지만, 몇 가지 예외적인 요청에는 전송.

**Lax 쿠키가 전송되는 경우**
[The Chromium Project docs](https://www.chromium.org/administrators/policy-list-3/cookie-legacy-samesite-policies/)를 보면 Lax를 다음과 같이 설명한다.
>A cookie with "SameSite=Lax" will be sent with a same-site request, or a cross-site top-level navigation with a "safe" HTTP method.

즉, 같은 사이트일때는 당연히 되고, top-level navigation(웹 페이지 이동)과, 안전한 HTTP method의 경우 전송된다는 것이다.

이 경우 유저가 링크(a tag)를 클릭하는 경우는 막지 않는다. 왜냐하면 top-level navigation이기 때문이다. 하지만 img태그 같은 요청은 top-level navigation이라고 할 수 없으니 막을 수 있다. 

또한 GET 요청과 같은 안전한 요청은 가능하고 POST, DELETE 같은 위험한 요청은 막는다.

크롬이나 파이어폭스, 엣지 등은 samesite에 대한 설정이 별도로 없으면 기존의 기본 값이던 None에서 Lax로 조정했다. 그래서 None으로 똑같이 유지하려면 https인증을 받고, 수동으로 None 설정을 해줘야한다.

spring에서 설정하는 방법은 다음과 같다.
```java
@Configuration
public class MvcConfiguration implements WebMvcConfigurer {
    @Bean
    public TomcatContextCustomizer sameSiteCookiesConfig() {
        return context -> {
            final Rfc6265CookieProcessor cookieProcessor = new Rfc6265CookieProcessor();
            cookieProcessor.setSameSiteCookies(SameSiteCookies.NONE.getValue());
            context.setCookieProcessor(cookieProcessor);
        };
    }
}
```
이외에도 웹 프록시 서버에서 일괄적으로 바꿔주거나 필터나 인터셉트에서 response를 가로채서 설정하는 방법 등 여러가지가 있다고 하니 필요하면 찾아보길 바란다. 


## 정리
spring security는 기본적으로 csrf 방어를 수행한다. 일부 글에서 csrf 방어 설정을 disable 시키라고 하는데 이는 MVC 패턴에서는 CSRF 노출 되는 것이다. MVC 패턴은 세션과 쿠키를 통해 사용자 인증을 수행하기 때문에 CSRF에 취약하다.

하지만 만약 api 서버를 만든다고 하면 보통 쿠키보다는 로컬 스토리지를, 세션보다는 jwt token을 이용하기 때문에 CSRF를 disable 하더라도 큰 상관이 없다. 


