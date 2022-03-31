# Cross-Origin Resource Sharing(CORS)에 대해 정확하게 알아보자

## 이 글을 쓰게 된 이유
프론트엔드 개발자건 백엔드 개발자건 웹 개발을 하다보면 CORS를 자주 접할 수 있다. 백엔드 개발자인 나는 단순하게 요청을 받을 수 있는 출처를 설정하는 것이라고만 추상적으로 정의하고 개발을 하고 있었다.
개념이 추상적으로 정의 되어있다보니 에러가 생겨 트러블 슈팅을 할 때 단순하게 해결 될 문제가 복잡해지고 제대로 해결도 안되는 문제가 발생하였다. **이 참에 CORS에 대해서 확실하게 정리해보자!**

## CORS가 왜 탄생했을까?
CORS 대해서 본격적으로 알아보기 전에 그 탄생 배경부터 알아보자. 이를 알기 위해서는 CORS의 반대말인 SOP(Same-Origin Policy)에 대해서 알아볼 필요가 있다.
### SOP(Same-Origin Policy)
SOP를 간단하게 정의하자면 **'다른 출처의 리소스를 사용하는 것을 제한하는 보안 방식'**이라고 할 수 있다. 풀어보자면 특정 출처의 리소스가 다른 출처의 리소스와 상호작용하는 것을 막겠다는 것이다. 그렇다면 여기에서 출처(origin)란 무엇일까?
![url구조](https://hanseul-lee.github.io/2020/12/24/20-12-24-URL/0.png)
위 사진은 url의 구조를 나타낸 것이다. 자세한 설명은 거두절미하고 우리가 알아보고 있는 출처(origin)만 얘기하겠다. 여기서 출처란 protocol+domain name(host)+port 이다. 즉, 이 세가지만 동일하면 브라우저에서는 동일한 출처로 인식한다는 것이다. (protocol, host, port에 대한 각각의 설명은 너무 길어지기 때문에 생략)

그렇다면 왜 SOP를 사용해야 보안에 도움이 될까? 예를 들어보자.
![해킹 예시](https://images.velog.io/images/younghoondoodoom/post/5561f4c6-bb00-471d-95d3-c5760871e399/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202022-03-25%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%201.58.33.png)
보통 CORS 해킹은 위와 같은 방식으로 진행 된다고 한다. 이를 방지하기 위해서 toss 앱에서는 3번을 막기 위해 SOP를 사용하여 다른 도메인에서 오는 요청을 막는다.

이렇게 SOP는 다른 출처라면 무조건적으로 차단함으로서 막강한 보안을 제공해준다. 하지만 **만약 다른 출처의 리소스가 필요**하다면 어떻게할까? 이러한 문제를 해결해주기 위해 **CORS**가 탄생했다.

## CORS란?
Cross-Origin Resource Sharing이라는 말 그대로 다른 출처의 리소스를 공유하겠다라는 뜻이다.
> 교차 출처 리소스 공유(Cross-Origin Resource Sharing, CORS)는 추가 **HTTP 헤더**를 사용하여, 한 출처에서 실행 중인 웹 애플리케이션이 다른 출처의 선택한 자원에 접근할 수 있는 권한을 부여하도록 **브라우저에 알려주는** 체제입니다. 웹 애플리케이션은 리소스가 자신의 출처(도메인, 프로토콜, 포트)와 다를 때 교차 출처 HTTP 요청을 실행합니다.
-mozilla-

여기에서 핵심은 출처가 다를 시에 http header를 이용해 브라우저에게 알려준다는 점이다!
## CORS 접근 제어 시나리오
 크게 preflight, simple reqeust, credentialed reqeust 세가지가 있는데 하나씩 알아보자.
### 1. Preflight
preflight란 클라이언트에서 서버로 요청을 보내기 전에 미리 허락(?)을 받는 방식을 얘기한다.
이는 OPTIONS 메소드를 이용해 다른 도메인의 리소스에 요청이 가능한지 확인하고 요청이 가능하다면 실제 요청(Actual Request)를 보내는 방식이다.
![](https://images.velog.io/images/younghoondoodoom/post/20d2a6cc-3a25-44d6-9606-358c28fb113d/image.png)
만약 불가능하다면 위와 같은 에러 메세지를 띄운다.
![](https://images.velog.io/images/younghoondoodoom/post/998288af-c2fc-4fa8-a587-4a227c77914c/image.png)
preflight request를 보낼 때 다음 3가지 헤더를 통해 서버에게 질문을 던진다. 
1. origin(출처)
클라이언트는 여기에 본인의 출처를 밝히고 이 출처가 허용된 출처인지 확인을 한다.
2. Access-Control-Request-Method
클라이언트는 여기에 본인이 요청을 보내는 http 메서드를 보낸다.(get, post 등) 서버는 이를 통해 이 도메인이 이 메서드를 허용하는지 확인을 한다. 
3.Acess-Control-Allow-Header
클라이언트는 여기에 실제 요청에 추가 헤더를 무엇을 보낼 수 있는지 물어보는 헤더이다.

이렇게 preflight request를 보내면 서버측에서 응답하는 preflight response 헤더는 다음과 같다.
1. Access-Control-Allow-Origin
서버에서 이 origin(출처)는 허가가 되어있다고 알려주는 헤더이다.
2. Access-Control-Allow-Methods
이 도메인에 대해 서버가 허용하는 http 메서드를 담아서 보내준다. 보통은 무조건 options 메서드를 포함한다.
3. Access-Control-Allow-Header
서버에서 허용이 되어있는 헤더를 보여준다.
4. Access-Control-Max-Age
Preflight 방식을 사용하면 요청을 두번 보내야해서 리소스를 많이 사용하게 된다. 이를 효율적으로 관리하고자 브라우저에서는 첫 요청에서 캐시를 만들어 저장하고 다음 번에 보낼 때는 캐시를 확인하고 본 요청 한번만 보내는 방식을 사용한다. Access-Control-Max-Age는 서버에서 브라우저에게 이 시간만큼 캐시를 생성하라는 명령이다. 보통 86400 초(24시간)를 보낸다.

또한 preflight response의 응답 코드는 200대여야만 하고 body는 비어있는 것이 좋다.

### 2. Simple Request
preflight 없이 simple request를 보내려면 다음과 같은 조건을 꼭 갖추어야한다.
1. 메소드는 GET, POST, HEAD 여야만 한다.
2. Content-Type은 application/x-www-form-urlencoded, multipart/form-data, text/plain 중 하나여야 한다.
3. 헤더는 Accept, Accept-Language, Content-Language, Content-Type 만 허용 된다.
![](https://images.velog.io/images/younghoondoodoom/post/0c50b255-3f66-4b98-9434-2a455e3d1317/image.png)
클라이언트는 자신의 origin을 보내고 서버에서는 Access-Control-Allow-Origin을 보내 확인을 한다. 만약 다르다면 당연히 cors 에러가 발생한다.

보다시피 simple request를 보내는 것이 preflight request를 보내는 것보다 훨씬 간편하다. **그렇다면 왜 preflight를 보내는 것일까??**
일단 cors error는 서버가 아닌 브라우저에서 내뱉는 에러이다. 그리고 브라우저에서 reponse를 받고 client에 뿌려주는 과정에서 발생하기 때문에 서버는 cors가 발생했는지 모른다. 이렇게 되면 출처가 불확실한 요청에 대해서 서버는 무방비로 요청을 받고 로직을 처리한다. 단순히 get요청 같은 경우는 상관없겠지만 delete나 put같은 요청을 받아 DB 수정이 일어날 가능성이 있다. 이러한 이유로 preflight request를 사용하는 것이다.

### Credentialed Reqeust
마지막으로 인증 관련된 헤더를 포함할 때 사용하는 credentialed reqeust가 있다. 만약 client 측에서 인증 쿠키나 jwt 토큰을 자동을 header에 담아서 보내고 싶을 때 사용된다. 

client에서는 credentials: include를 설정한다.
server에서는 Access-Control-Allow-Credential: True 로 설정을 해야한다. (Access-Controtl-Allow-Origin: * 은 불가능하다. 구체적인 origin을 설정을 해줘야한다.)

## CORS를 해결해보자!
CORS가 무엇인지 알았다면 CORS를 해결해보자. 
알아보니 몇가지 방법들이 있다. 여기서 내가 소개할 방법은 프론트의 프록시 설정을 바꾸는 것이다. 다른 방법으로는 직접 헤더를 설정해주기(복잡해보임.), 프레임워크의 기능을 이용하기(보통 현업에서 많이 쓰임.) 등이 있다.

### 프론트 프록시 서버 설정 바꿔서 CORS 피하기
![](https://images.velog.io/images/younghoondoodoom/post/0e3cc8f9-0fa0-42a4-b65b-47fa98fb37b5/image.png)

보통 우리는 배포를 할 때 NGINX 같은 웹 서버를 브라우저 앞에다가 둔다. 웹 서버(프록시 서버)의 설정을 바꾸어서 CORS를 간단하게 해결할 수 있다. 위 그림에서 보듯이 브라우저에서 웹 서버에 요청을 하면 origin과 target이 같기 때문에 same origin으로 인식해 문제가 발생하지 않는다. 이때 웹 서버에서 특정 도메인(뒤에 /api가 붙어있는 등)이 들어오면 target port를 바꿔준다. 이렇게 간단하게 해결이 가능하다.

보통 현업에서는 해당 프레임워크가 가지고 있는 기능을 이용해 해결한다고 한다.(ex->spring boot) 본인이 쓰는 프레임워크의 기능들을 확인하여 해결하자!

