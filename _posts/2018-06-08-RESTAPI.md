---
title: REST API에 대해서
author: wcm
layout: post
---

●REST API
2000년도에 로이 필딩이 웹(HTTP)의 우수성에 비해 제대로 사용되지 못하는 안타까움에, 웹의 장점을 최대한 활용할 수 있는 아키텍쳐로써 발표.


●특징
(1) 유니폼 인터페이스(Uniform Interface)
유니폼 인터페이스는 *URI로 지정한 리소스에 대한 조작을, 통일되고 한정적인 인터페이스로 수행하는 아키텍쳐 스타일
(URI(Uniform Resouce Identifier)는 통합 자원 식별 지원자로 인터넷에 있는 자원을 나타내는 유일한 주소이다. URL은 URI의 하위 항목이다.

(2) 무상태성(Stateless)
REST는 작업을 위한 상태 정보를 따로 저장하고 관리하지 않는, 무상태성 성격을 갖는다. 세선이나 쿠키 정보를 별도로 저장하고 관리하지 않기 때문에 API 서버는 들어오는 요청만을 처리한다. 그렇기 때문에 서비스의 자유도가 높고, 서버에서 불필요한 정보를 관리하지 않아 구현이 단순해진다.

(3) 캐시 가능(Cacheable)
REST는 HTTP라는 기존 웹 표준을 그대로 사용하기 때문에, 웹에서 사용하는 기존 인프라를 그대로 활용이 가능하다. 따라서 HTTP가 가진 캐싱 기능이 적용 가능하다. HTTP 프로토콜 표준에서 사용하는 Last-Modified 태그나 E-Tag를 이요하면 캐싱 구현이 가능하다.

(4) 자체 표현 구조(Self-descriptiveness)
REST의 또 다른 큰 특징 중 하나는 REST API 메시지만 보고도 이를 쉽게 이해할 수 있는 자체 표현 구조로 표현된느 것이다.

(5) Client-Server 구조
REST 서버는 API 제공, 클라이언트는 사용자 인증이나 컨텍스트(세션, 로그인 정보) 등을 직접 관리하는 구조로 각각의 역할이 확실히 구분되기 때문에 클라이언트와 서버에서 개발해야 할 내용이 명확해지고 서로간 의존성이 줄어들게 된다.

(6) 계층형 구조
REST 서버는 다중 계층으로 구성될 수 있으며 보안, 로드 밸런싱, 암호화 계층을 추가해 구조상의 유연성을 둘 수 있고, PROXY, 게이트웨이 같은 네트워크 기반의 중간 매체를 사용할 수 있게 한다.


●REST API 디자인 가이드

(1) REST API의 URI 정보는 자원을 표현해야 한다. (리소스 명은 동사보다는 명사를 사용)
-예 : GET /members/delete/1 (X) : delete같은 행위에 대한 표현이 들어가면 안됨


(2) 자원에 대한 행위는 HTTP Method (GET, POST, PUT, DELETE 등)로 표현
-예 : DELETE /members/1
-회원 정보를 가져오는 URI 예 : GET /members/show/1 (X)
                       GET /members/1      (O)
-회원을 추가하는 예 : GET /members/insert/2 (X)
               POST /members/2       (O)

			   
(3) URI 설계 시 주의 사항

(3)(1) 슬래시 구분자(/)는 계층 관계를 나타내는데 사용한다.
-예 : http://restapi.example.com/houses/apartments
      http://restpai.example.com/animals/mammals/whales

(3)(2) URI 마지막 문자로 슬래시(/)를 포함하지 않는다.
-예 : http://restapi.example.com/houses/apartments/ (X)

(3)(3) 하이픈(-)은 URI 가독성을 높이는데 사용

(3)(4) 밑줄(_)은 URI에 사용하지 않는다. (글꼴에 따라 보기 어렵거나 문자가 가려지기도 하기 때문)

(3)(5) URI 경로에는 소문자가 적합하다. (RFC 3986(URI 문법 형식)은 URI 스키마와 호스트를 제외하고 대소문자에 따라 다른 리소스로 인식하기 때문)

(3)(6) 파일 확장자는 URI에 포함시키지 않는다. Accept header를 사용한다.
-예 : http://restapi.example.com/members/soccer/345/photo.jpg (X)
-예 : GET /members/soccer/345/photo HTTP/1.1 Host : restapi.example.com.Accept: image/jpg


(4) 리소스 간의 관계를 표현하는 방법
/리소스명/리소스 ID/관계까 있는 다른 리소스명
-예 : GET : /users/{userId}/devices (일반적으로 소유 'has'의 관계를 표현할 때)
관계명이 복잡하다면 서브 리소스에 명시적으로 표현할 수 있다.
-예 : 사용자가 '좋아하는' 디바이스 목록을 표현할 경우 GET : /users/{userId}/likes/devices (관계명이 애매하거나 구체적인 표현이 피룡할 때)


(5) 자원을 표현하는 Collection과 Document

Document는 단순히 문서로 이해해도 되고, 한 객체라고 이해해도 된다. Collection은 문서들의 집합, 객체들의 집합이라고 생각하면 이해하는데 도움이 된다. Collection과 Document는 모두 리소스라고 표현할 수 있으며 URI에 표현된다.
-예 : sports라는 컬렉션과 soccer라는 도큐먼트 http:// restapi.example.com/sports/soccer
-예 : sports,players 컬렉션과 soccer,13 도큐먼트의 URI http:// restapi.example.com/sports/soccer/players/13 (컬렉션의 경우 복수를 사용함)

