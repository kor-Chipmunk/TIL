# 식별, 인가, 보안 ( 미완성 )

## 학습 목표

본 문서에 다음의 내용이 있다.

1. 쿠키로 사용자를 식별해 콘텐츠를 개인화시키는 기법을 다룬다.
2. 기본적인 인증 방법으로 어떻게 사용자를 확인하는지를 다룬다. 그리고 어떻게 HTTP 인증이 데이터베이스와 동작하는지도 알아본다.
3. 다이제스트 인증 방법이 무엇이고 어떻게 보안을 향상 시켰는지를 다룬다.
4. 인터넷 암호화 기법이 무엇인지, 디지털 서명은 무엇인지, 보안 소켓 계층(Secure Sockets Layer, SSL)이란 무엇인지 알아본다.

## 목차

1. [클라이언트 식별과 쿠키](#클라이언트-식별과-쿠키)
2. [기본 인증](#기본-인증)
3. [다이제스트 인증](#다이제스트-인증)
4. [보안 HTTP](#보안-HTTP)

# 클라이언트 식별과 쿠키

웹 서버는 수 많은 클라이언트와 동시에 통신한다. 알 수 없는 상대방의 요청을 처리하는 것도 중요하지만, 몇 번 만난 적 있는 상대방을 기억해야 할 때도 많다. 서버가 통신하는 대상을 식별하는 데 사용하는 기술을 알아보자.

## 목차

1. [개별 접촉](#1-개별-접촉)
2. [HTTP 헤더](#2-HTTP-헤더)
3. [클라이언트 IP 주소](#3-클라이언트-IP-주소)
4. [사용자 로그인](#4-사용자-로그인)
5. [뚱뚱한 URL](#5-뚱뚱한-URL)

## 1. 개별 접촉

HTTP의 특성은 다음과 같다.

- 익명으로 사용하고 상태가 없다.
  - 연결 자체에 대한 정보를 가지지 않고 매 요청은 독립적이고 일회성이다. Stateless 라고 한다.
- 요청과 응답으로 통신하는 프로토콜

웹 서버는 요청을 보낸 사용자를 식별하거나 방문자가 보낸 연속적인 요청을 따라가려고 정보를 더 이용할 수도 있다. 현대의 웹 사이트들은 개인화된 서비스를 제공하려고 한다. 네트워크로 연결된 사용자들을 더 알고 싶어 하고 사용자들의 브라우징 기록을 하고 싶어 한다. 이미 유명한 쇼핑 사이트는 여러 가지 방법으로 사이트를 개인화 시켜 제공한다.

### 개별 인사

개인에게 맞춰져 있는 것처럼 느끼게 하려고 사용자에게 특화된 환영 메시지나 페이지 내용을 만든다.

### 사용자 맞춤 추천

온라인 상점은 고객의 흥미가 무엇인지 학습해서 취향에 맞는 제품들을 추천하려고 한다. 생일이나 다른 중요한 날이 다가오면 특별한 상품을 제시하기도 한다.

### 저장된 사용자 정보

고객은 매 번 입력하는 것을 싫어한다. 한 번 고객을 식별하고 나면, 더 편하게 서비스를 이용할 수 있도록 저장된 사용자 정보를 사용할 수 있다.

### 세션 추적

HTTP 트랜잭션은 상태가 없다. 각 요청과 응답은 독립적으로 일어난다. 사용자와 사이트가 상호작용할 수 있도록 사용자의 상태를 남긴다. 그 상태를 유지하려면, 각 사용자에게 오는 HTTP 트랜잭션을 식별해야 한다.

---

사용자를 식별하는 방법은 여러 개 있다.

- 사용자 식별 관련 정보를 전달하는 HTTP 헤더들
- 클라이언트 ip 주소 추적으로 알아낸 ip 주소로 사용자를 식별
- 사용자 로그인 인증을 통한 사용자 식별
- URL에 식별자를 포함하는 기술인 Fat URL
- 식별 정보를 지속해서 유지하는 강력하고 효율적인 기술인 쿠키

## 2. HTTP 헤더

사용자 정보를 전달하는 일반적인 7가지 HTTP 요청 헤더는 다음과 같다.

|헤더 이름|헤더 타입|설명|
|---|---|---|
|From|요청|사용자의 이메일 주소|
|User-Agent|요청|사용자의 브라우저|
|Referer|요청|사용자가 현재 링크를 타고 온 근원 페이지|
|Authorization|요청|사용자 이름과 비밀번호|
|Client-ip|확장(요청)|클라이언트의 IP 주소|
|X-Forwared-For|확장(요청)|클라이언트의 IP 주소|
|Cookie|확장(요청)|서버가 생성한 ID 라벨|

From 헤더로 사용자를 식별할 수 있었지만, 악의적인 서버가 이메일 주소를 모아 스팸 메일을 발송하는 문제가 있기에 From 헤더를 보내는 브라우저는 많지 않다. 데이터를 수집하는 로봇이나 스파이더는 그 과정에서 실수로 웹 사이트에 문제를 일으켰을 때, 해당 사이트의 운영자가 항의 메일을 보낼 수 있도록 From 헤더에 이메일 주소를 기술하기도 한다.

User-Agent 헤더는 사용자가 쓰고 있는 브라우저의 이름과 버전 정보, 운영체제 같은 정보까지 포함해 서버에게 알려준다. 특정 브라우저에서 제대로 동작하도록 콘텐츠를 최적화하는데 유용할 수도 있다. 그러나 사용자를 식별하는 데는 큰 도움은 되지 않는다.

Referer 헤더는 현재 페이지로 유입하게 한 웹페이지의 URL을 가리킨다. 이전에 어떤 페이지를 방문했는지 알려준다. 이 헤더로 사용자의 웹 사용 형태나 사용자의 취향을 더 잘 파악할 수 있게 된다.

## 3. 클라이언트 IP 주소

웹 초기는 사용자 식별에 클라이언트의 IP 주소를 사용하려고 했다. 클라이언트 IP 주소는 보통 표준 HTTP 헤더엔 없지만 웹 서버는 HTTP 요청을 보내는 반대쪽 TCP 커넥션의 IP 주소를 알아낼 수 있다.

다만 클라이언트 IP 주소로 사용자를 식별하는 방식은 다음과 같은 약점이 있다.

- 클라이언트 IP 주소는 사용자가 아닌, 사용하는 기기를 가리킨다. 기기가 여러 개 라면 정확한 사용자를 식별할 수 없다.
- 대부분 인터넷 서비스 제공자(ISP)는 사용자가 로그인할 때 동적으로 IP 주소를 할당해준다. 시간에 따라 IP 주소가 달라질 수 있으므로, 웹 서버는 정확하게 식별한다는 보장이 없어진다.
- 보안을 강화하고 싶은 많은 사용자가 네트워크 주소 변환(Network Address Translation, NAT) 방화벽으로 인터넷을 사용한다. NAT 장비들은 클라이언트의 실제 IP 주소를 방화벽 뒤로 숨기고, 클라이언트의 실제 IP 주소를 내부에서 사용하는 하나의 방화벽 IP 주소와 포트로 변환한다.
- HTTP 프록시와 게이트웨이는 원래 서버에 새로운 TCP 연결을 한다. 웹 서버는 클라이언트의 IP 주소가 아닌 프록시 서버의 IP 주소를 보게 된다. 프록시는 실제 클라이언트 IP 주소를 전달하는 확장 헤더(Client-ip, X-Forwareded-For HTTP)를 전달해주기도 하지만 모든 프록시가 제공해주진 않는다.

IP 주소를 활용하는 방법은 보안 기능으로 지정된 IP 주소에게만 열람이 가능하도록 해주는 방법이 있다. 인트라넷 같이 제한된 영역에서는 적절할 수 있지만 인터넷 상황에서는 IP 주소를 임의로 변경할 수 있기에 문제가 발생할 수 있다. 심지어 IP 주소로 문서의 접근 권한을 정하는 방법보다 더 좋은 방법이 있다.

## 4. 사용자 로그인

웹 서버는 사용자 이름과 비밀번호로 인증할 것을 요구해 명시적으로 식별 요청을 할 수 있다. 웹사이트 로그인이 쉽도록 HTTP는 `WWW-Authenticate`와 `Authorization` 헤더를 사용해 웹 사이트에 사용자 이름을 전달하는 체계도 있다.

한 번 로그인 하면 브라우저는 사이트로 보내는 모든 요청에 이 로그인 정보를 함께 보내므로 웹 서버는 로그인 정보를 항상 확인할 수 있다. 이 방식은 다음과 같이 동작한다.

1. 사용자가 사이트에 접근할 때 `HTTP 401 Login Required` 응답 코드와 `WWW-Authenticate`를 브라우저에 보낸다.
2. 브라우저는 로그인 대화상자를 보여준다.
3. 다음 요청부터 `Authorization` 헤더에 사용자의 식별정보 토큰을 같이 보낸다.
   1. 언제까지? 한 세션이 유지될 때 까지

웹 사이트 로그인은늘 귀찮다. 사이트를 옮길 때 마다 로그인을 해야만 한다면, 지루한 경험이 될 것 이다.  
아이디와 비밀번호로 로그인을 하지 않고 어떻게 하면 웹 서버에서 사용자를 식별하고 추적할 수 있을까?

## 5. 뚱뚱한 URL

뚱뚱한 URL은 사용자의 상태 정보를 포함하고 있는 URL을 의미한다. 웹 서버와 통신하는 독립적인 HTTP 트랜잭션을 하나의 세션, 방문으로 묶을 수 있다.  
웹 서버에 처음 접속했을 때 유일한 아이디를 부여 받고 서버가 사용자를 인식할 수 있게 모든 하이퍼링크 URL에 추가가 된다.  
그러나 다음과 같은 심각한 단점들이 있다.

- 못생긴 URL : 새로운 사용자에게 혼란을 줄 수 있다.
- 공유하지 못하는 URL : 주소를 공유하면, 쌓인 개인 정보를 공유하게 되는 셈이 된다.
- 캐시를 사용할 수 없음 : URL이 달라지기 때문에 기존 캐시에 접근할 수 없다.
- 서버 부하 가중 : 뚱뚱한 URL에 해당하는 HTML 페이지를 다시 그려야 한다.
- 이탈 : 뚱뚱한 URL 세션에서 이탈할 확률이 크다. 쌓인 개인 정보는 다시 사라지게 된다. 
- 세션 간 지속성의 부재 : 그 당시 정보가 담긴 URL를 저장하지 않는다면, 로그아웃하면 모든 정보를 잃게 된다.

## 6. 쿠키

쿠키는 현재까지 가장 널리 사용하는 방식이다. 앞서 설명한 기술들이 가지고 있는 문제점들을 해결했다. 그러나 쿠키만으로 하기 힘든 일도 분명 있다. 넷스케이프가 최초로 개발했지만 모든 브라우저에서 지원한다. 새로운 HTTP 헤더를 정의하므로, 더 자세히 볼 필요가 있다.

쿠키는 캐시와 충돌할 우려가 있다. 따라서 캐시 대부분과 브라우저는 쿠키에 있는 내용물을 캐싱하지 않는다.

### 쿠키의 타입

쿠키는 세션 쿠키와 지속 쿠키로 나눈다.
- 세션 쿠키 : 사용자가 사이트를 탐색할 때, 관련 설정과 선호 사항들을 저장하는 임시 쿠키. 브라우저를 닫으면 삭제된다.
- 지속 쿠키 : 디스크에 저장돼, 브라우저를 닫거나 컴퓨터를 재시작해도 남아있다. 주기적으로 방문하는 사이트에 대한 설정 정보나 로그인 이름을 유지하려고 사용한다.

둘의 차이점은 사라지는 시점이다. `Discard` 파라미터가 설정됐거나, 파기 시점을 가리키는 `Expires` 또는 `Max-Age` 파라미터가 없으면 세션 쿠키가 된다.

# 기본 인증

## 목차

1. [개별 접촉](#1-개별-접촉)
2. [HTTP 헤더](#2-HTTP-헤더)
3. [클라이언트 IP 주소](#3-클라이언트-IP-주소)
4. [사용자 로그인](#4-사용자-로그인)
5. [뚱뚱한 URL](#5-뚱뚱한-URL)

# 다이제스트 인증

## 목차

1. [개별 접촉](#1-개별-접촉)
2. [HTTP 헤더](#2-HTTP-헤더)
3. [클라이언트 IP 주소](#3-클라이언트-IP-주소)
4. [사용자 로그인](#4-사용자-로그인)
5. [뚱뚱한 URL](#5-뚱뚱한-URL)

# 보안 HTTP

## 목차

1. [개별 접촉](#1-개별-접촉)
2. [HTTP 헤더](#2-HTTP-헤더)
3. [클라이언트 IP 주소](#3-클라이언트-IP-주소)
4. [사용자 로그인](#4-사용자-로그인)
5. [뚱뚱한 URL](#5-뚱뚱한-URL)

---

# 참고 도서

HTTP 완벽 가이드