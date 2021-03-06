# HTTP와 HTTPS

## 이 글을 쓰게 된 이유
프로젝트를 진행하며 웹캠을 사용할 일이 있었다. 서치를 해보니 웹캡을 사용하려면 꼭 SSL인증서를 받아서 HTTPS를 이용해야한다는 정보를 얻었다. 기존에도 HTTPS가 대충 무엇인지는 알았으나 정확히는 몰랐다. 그러다보니 직접 구현하는데에 어려움을 겪었고 이 참에 정확하게 정리하고자 이 글을 쓰게되었다.

## HTTP란?
HyperText Transfer Protocol의 약자이며 클라이언트와 서버 사이에 이루어지는 요청/응답(request/response) 프로토콜이다. 여기에서 HyperText란 HTML로 생각하면 된다. 즉, HTTP란 Hypertext 인 HTML을 전송하기 위한 통신규약을 의미한다. 요청을 보낼 때 호스트 앞에 http가 있는 것을 많이 봤을 텐데 이는 해당 도메인으로 요청하는데 http 방식으로 요청한다는 뜻이다. 그럼 클라이언트와 서버 모두 http방식으로 요청과 응답을 해석한다.

## HTTPS와 SSL
HTTPS에서 S는 Secure의 약자이다. 뭔지는 몰라도 HTTP 통신에서 조금 더 안전하다는 것을 알 수 있다. 그렇다면 HTTPS는 HTTP에서 어떤 부분을 보안할 것일까?

HTTP로 로그인을 한다고 생각해보자. 사용자는 Client측에 아이디와 비밀번호를 적고 Server로 전송할 것이다. 이때 전송하는 데이터는 아무런 암호화도 거치지 않고 그대로 전송하게 된다. 즉, 중간에 누군가가 중간에 데이터를 채가면 사용자의 개인정보는 그대로 노출되는 것이다.

HTTPS는 이와 같은 문제를 해결하고자 Client에서 Server로 요청할 때 데이터를 암호화해서 보낸다. 중간에 누군가가 데이터를 채가도 무슨 데이터인지 모르게 만드는 것이다. Server에서는 그 데이터를 미리 약속된 복호화 알고리즘을 통해서 원본 데이터로 만들어서 사용하게 된다.

그렇다면 무슨 규칙으로 암호화, 복호화를 하는 것일까? 이를 규정하고 인증하는 것이 SSL이다.
![](https://images.velog.io/images/younghoondoodoom/post/8d0c9c55-2b91-4d15-bb13-4bf6250b9ca8/image.png)
위 사진은 SSL의 계층을 보여준다. 사진에서도 볼 수 있듯이 SSL(TLS는 SSL와 같다고 보면 된다.)위에서 여러 프로토콜이 작동한다.

## SSL 인증서
SSL 인증서란 클라이언트와 서버간의 통신을 제3자가 보증해주는 전자화된 문서다. 클라이언트가 서버에 접속한 직후에 서버는 클라이언트에게 이 인증서 정보를 전달한다. 클라이언트는 이 인증서 정보가 신뢰할 수 있는 것인지를 검증 한 후에 다음 절차를 수행하게 된다. SSL과 SSL 디지털 인증서를 이용했을 때의 이점은 아래와 같다.
- 통신 내용이 공격자에게 노출되는 것을 막을 수 있다. 
- 클라이언트가 접속하려는 서버가 신뢰 할 수 있는 서버인지를 판단할 수 있다.
- 통신 내용의 악의적인 변경을 방지할 수 있다.
From 생활코딩 이고잉님

그렇다면 어떻게 통신 내용이 공격자에게 노출되는 것을 막을 수 있을까? 바로 암호화, 복호화이다. 이 부분에 대해서 자세하게 알아보자

### 암호화, 복호화
![](https://images.velog.io/images/younghoondoodoom/post/b50f5732-13eb-44ed-9703-a8264c01727a/image.png)
위 사진은 전체적인 암호화와 복호화의 흐름을 보여준다. 암호화를 하려면 암호화 키가 필요하고 복호화를 하려면 복호화 키가 필요하다. 키가 없는 사람은 어떤 내용인지 모르게 하는 것이 목적이다. 
#### 대칭키
대칭키 방식은 암호화하는 쪽과 복호화하는 쪽이 모두 같은 키를 가지고 있어야한다. 즉, 그 키로 암호화와 복호화를 모두 하는 것이다. 이 키만 가지고 있으면 누구든지 암호화와 복호화를 할 수 있다. 
굉장히 편리하고 안전한 방식이다. 하지만 대칭키에는 치명적인 단점이 있다. 암호를 주고 받는 사람들 사이에 대칭키를 전달하는 것이 어렵다는 점이다. 최초로 한 쪽에서 키를 다른 쪽으로 전달할 때 그 키는 암호화되지 않는 상태로 전달이 된다. 만약 누군가가 그 키를 갖게 될 수도 있다는 것이다.

#### 공개키
공개키는 대칭키의 단점을 해결하고자 나온 방식이다.
공개키 방식은 공개키(Public key)와 비밀키(Private key) 두개를 갖는다. 공개키와 비밀키의 관계는 공개키로 암호화한 정보를 비밀키로 복호화할 수 있고, 비밀키로 암호화한 정보를 공개키로 암호화할 수 있다. 이게 끝이다.

그렇다면 어떻게 공개키 방식으로 대칭키의 단점을 보안할까?
Server에서 공개키와 비밀키를 가지고 있다. Client에서 누구나 소유할 수 있는 공개키를 가지고 데이터(정보)를 암호화해서 보낸다. 그럼 Server는 비밀키를 이용해 그 정보를 복호화한다.
즉, 키 전달을 할 필요가 없어진다! 

또한 공개키를 이렇게 응용할 수도 있다. 비공개키 소유자(Server)는 비공개키를 이용해 데이터(정보)를 암호화해서 통신하고 싶은 곳(Client)에 보낸다. 그럼 누구나 소유할 수 있는 공개키를 가지고 있는 쪽(Client)에서는 공개키를 통해 복호화를 한다. 이렇게 되면 비공개키 소유자가 비공개키를 가졌다는 것을 알 수 있다! 그럼 비공개키 소유자는 신뢰할 수 있게 되는 것이다. 이를 **전자서명** 이라고 부른다.
꼭 Server와 Client가 아니어도 좋다. 예를 들기 위함이다.

## SSL 인증
공개키의 개념을 알았다면 SSL 인증서의 기능을 이해하는 것은 쉽다.
인증서의 기능은 크게 2가지다.
- 클라이언트가 접속한 서버가 신뢰 할 수 있는 서버임을 보장한다.
- SSL 통신에 사용할 공개키를 클라이언트에게 제공한다.

위와 같은 기능들을 제공해야하기 때문에 SSL 인증서는 신뢰도가 상당히 높아야한다. 그렇기 때문에 브라우저마다 공인된 사기업들을 선정해서 이 기업들이 발행한 SSL 인증서만 인정한다. 그러한 기업들을 CA(Certificate authority)라고 부른다. 
CA는 각 브라우저에 검색하면 쉽게 알 수 있다.

### SSL 인증서의 내용
SSL 인증서의 내용은 크게 2가지이다.
> 	1. 서비스의 정보 (인증서를 발급한 CA, 서비스의 도메인 등등)
	2. 서버 측 공개키 (공개키의 내용, 공개키의 암호화 방법)

1번은 클라이언트가 접속한 서버가 클라이언트가 의도한 서버가 맞는지에 대한 내용을 담고 있고, 2번은 서버와 통신을 할 때 사용할 공개키와 그 공개키의 암호화 방법들의 정보를 담고 있다. **서비스의 도메인, 공개키와 같은 정보는 서비스가 CA로부터 인증서를 구입할 때 제출해야 한다.** 

CA는 제출 받은 정보를 토대로 검증을 한 후 CA의 비공개키를 이용하여 서버가 제출한 인증서를 암호화 한다. CA의 비공개키는 절대로 유출되어서는 안된다. 이것이 노출되는 바람에 디지노타라는 회사는 파산된 사례도 있다.

CA를 브라우저는 알고 있다
인증서를 이해하는데 꼭 알고 있어야 하는 것이 CA의 리스트다. 브라우저는 내부적으로 CA의 리스트를 미리 파악하고 있다. 이 말은 브라우저의 소스코드 안에 CA의 리스트가 들어있다는 것이다. 브라우저가 미리 파악하고 있는 CA의 리스트에 포함되어야만 공인된 CA가 될 수 있는 것이다. CA의 리스트와 함께 각 CA의 공개키를 브라우저는 이미 알고 있다.


### 인증서가 서버의 신뢰성을 제공하는 방법 
SSL인증서 자체도 검증이 필요하다. 이 인증서가 공인된 CA로 부터 발급된 것이어야만 하기 때문이다. 이 검증은 서버의 신뢰성 또한 제공한다. 왜냐하면 그 서버는 인증된 CA로부터 인증서를 받았기 때문이다.

다음과 같은 플로우로 브라우저는 인증서의 신뢰도를 검증한다.

1. 웹 브라우저가 서버에 접속할 때 서버는 제일 먼저 인증서를 제공(보통 Nginx나 Apache서버에서 인증서 경로를 설정해놓는다.)
2. 브라우저는 이 인증서를 발급한 CA가 자신이 내장한 CA의 리스트에 있는지를 확인
3. 확인 결과 서버를 통해서 다운받은 인증서가 내장된 CA 리스트에 포함되어 있다면 해당 CA의 공개키를 이용해서 인증서를 복호화
4. 복호화가 성공한다는 의미는 이 인증서가 CA의 비밀키로 복호화 되어있음을 의미하고 그렇다면 검증 성공(전자서명 됐다고 한다)

그렇다면 인증서에 포함된 공개키는 어디에 사용될까?


### SSL의 동작 방법
가장 안전한 방법은 그냥 공개키 방식을 사용하는 것이다. 
클라이언트와 서버가 각각 공개키와 비밀키를 가지고 있고, 각각의 공개키를 서로에게 전송한다. 그러면 데이터를 보낼때 비밀키로 암호화하고 전송받은 공개키로 복호화하면 된다.
하지만 이 방식은 성능적으로 떨어지는 방식이다. 그렇기 때문에 SSL은 대칭키와 공개키를 혼합해서 사용하는 방식을 사용하고 있다. 이 방식에 대해서 알아보자!

클라이언트와 서버가 주고 받는 실제 정보는 대칭키 방식으로 암호화하고, 대칭키 방식으로 암호화된 실제 정보를 복호화할 때 사용할 대칭키는 공개키 방식으로 암호화해서 클라이언트와 서버가 주고 받는다. 이 설명만으로는 이해하기 어려울 것이다. 아래의 관계만 일단 머리속에 기억해두고 좀 더 구체적인 설명으로 넘어가자.
- 실제 데이터 : 대칭키
- 대칭키의 키 : 공개키

컴퓨터와 컴퓨터가 네트워크를 이용해서 통신을 할 때는 내부적으로 3가지 단계가 있다. 아래와 같다.
악수 -> 전송 -> 세션종료

이 단계에 대해서 순차적으로 알아보자.

#### 1. Handshake(악수)
사람과 사람이 악수를 할 때처럼 서로의 상태를 살피는 단계이다. 즉, Client와 Server 간의 신뢰도 및 상태를 확인하는 단계라고 보면 된다.
SSL 방식을 이용해서 통신을 하는 브라우저와 서버 역시 핸드쉐이크를 하는데, 이 때 SSL 인증서를 주고 받는다. 이 과정은 위에서 설명했다. 이제 SSL 인증서에 있는 공개키가 무슨 일을 하는지 순차적으로 알아보자.
밑에 자료는 유튜브 생활코딩 이고잉님의 사이트인 https://opentutorials.org/course/228/4894 를 그대로 가져온 것이다.

1. 클라이언트가 서버에 접속한다. 이 단계를 Client Hello라고 한다. 이 단계에서 주고 받는 정보는 아래와 같다.
클라이언트 측에서 생성한 랜덤 데이터 : 아래 3번 과정 참조
클라이언트가 지원하는 암호화 방식들 : 클라이언트와 서버가 지원하는 암호화 방식이 서로 다를 수 있기 때문에 상호간에 어떤 암호화 방식을 사용할 것인지에 대한 협상을 해야 한다. 이 협상을 위해서 클라이언트 측에서는 자신이 사용할 수 있는 암호화 방식을 전송한다.
세션 아이디 : 이미 SSL 핸드쉐이킹을 했다면 비용과 시간을 절약하기 위해서 기존의 세션을 재활용하게 되는데 이 때 사용할 연결에 대한 식별자를 서버 측으로 전송한다.
 
2. 서버는 Client Hello에 대한 응답으로 Server Hello를 하게 된다. 이 단계에서 주고 받는 정보는 아래와 같다.
- 서버 측에서 생성한 랜덤 데이터 : 아래 3번 과정 참조
- 서버가 선택한 클라이언트의 암호화 방식 : 클라이언트가 전달한 암호화 방식 중에서 서버 쪽에서도 사용할 수 있는 암호화 방식을 선택해서 클라이언트로 전달한다. 이로써 암호화 방식에 대한 협상이 종료되고 서버와 클라이언트는 이 암호화 방식을 이용해서 정보를 교환하게 된다.
- 인증서
 
3. 클라이언트는 서버의 인증서가 CA에 의해서 발급된 것인지를 확인하기 위해서 클라이언트에 내장된 CA 리스트를 확인한다. CA 리스트에 인증서가 없다면 사용자에게 경고 메시지를 출력한다. 인증서가 CA에 의해서 발급된 것인지를 확인하기 위해서 클라이언트에 내장된 CA의 공개키를 이용해서 인증서를 복호화한다. 복호화에 성공했다면 인증서는 CA의 개인키로 암호화된 문서임이 암시적으로 보증된 것이다. 인증서를 전송한 서버를 믿을 수 있게 된 것이다.
클라이언트는 상기 2번을 통해서 받은 서버의 랜덤 데이터와 클라이언트가 생성한 랜덤 데이터를 조합해서 pre master secret라는 키를 생성한다. 이 키는 뒤에서 살펴볼 세션 단계에서 데이터를 주고 받을 때 암호화하기 위해서 사용될 것이다. 이 때 사용할 암호화 기법은 대칭키이기 때문에 pre master secret 값은 제 3자에게 절대로 노출되어서는 안된다.
그럼 문제는 이 pre master secret 값을 어떻게 서버에게 전달할 것인가이다. 이 때 사용하는 방법이 바로 공개키 방식이다. 서버의 공개키로 pre master secret 값을 암호화해서 서버로 전송하면 서버는 자신의 비공개키로 안전하게 복호화 할 수 있다. 그럼 서버의 공개키는 어떻게 구할 수 있을까? 서버로부터 받은 인증서 안에 들어있다. 이 서버의 공개키를 이용해서 pre master secret 값을 암호화한 후에 서버로 전송하면 안전하게 전송할 수 있다.

4. 서버는 클라이언트가 전송한 pre master secret 값을 자신의 비공개키로 복호화한다. 이로서 서버와 클라이언트가 모두 pre master secret 값을 공유하게 되었다. 그리고 서버와 클라이언트는 모두 일련의 과정을 거쳐서 pre master secret 값을 master secret 값으로 만든다. master secret는 session key를 생성하는데 이 session key 값을 이용해서 서버와 클라이언트는 데이터를 대칭키 방식으로 암호화 한 후에 주고 받는다. 이렇게해서 세션키를 클라이언트와 서버가 모두 공유하게 되었다는 점을 기억하자.

#### 2. Session(세선)
세션은 실제로 서버와 클라이언트가 데이터를 주고 받는 단계이다. 이 단계에서 핵심은 정보를 상대방에게 전송하기 전에 session key 값을 이용해서 대칭키 방식으로 암호화 한다는 점이다. 암호화된 정보는 상대방에게 전송될 것이고, 상대방도 세션키 값을 알고 있기 때문에 암호를 복호화 할 수 있다.

#### 3. Session exit(세션 종료)
데이터의 전송이 끝나면 SSL 통신이 끝났음을 서로에게 알려준다. 이 때 통신에서 사용한 대칭키인 세션키를 폐기한다.

실제 사용 방법은 CA에서 인증서를 구입하고 웹서버(Nginx, Apache)로 인증서 경로를 설정한다. 자세한 방법은 각 CA와 웹서버의 공식 문서를 통해 습득할 수 있다. 





