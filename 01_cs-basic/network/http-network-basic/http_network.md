## 제 1장. 웹과 네트워크의 기본에 대해 알아보자
* 1.1 웹은 HTTP로 나타낸다
* 1.2 HTTP는 이렇게 태어났고 성장했다
  * 1.2.1 웹은 지식 공유를 위해 고안되었다
  * 1.2.2 웹이 성장한 시대
  * 1.2.3 진보 안하는 HTTP
* 1.3 네트워크의 기본은 TCP/IP
  * 1.3.1 TCP/IP는 프로토콜의 집합
  * 1.3.2 계층으로 관리하는 TCP/IP
  * 1.3.3 TCP/IP 통신의 흐름
* 1.4 HTTP와 관계가 깊은 프로토콜은 IP/TCP/DNS
  * 1.4.1 배송을 담당하는 IP
  * 1.4.2 신뢰성을 담당하는 TCP
* 1.5 이름 해결을 담당하는 DNS
* 1.6 이들과 HTTP와의 관계
* 1.7 URI와 URL
  * 1.7.1 URI는 리소스 식별자
  * 1.7.2 URL 포맷

## 제 2장. 간단한 프로토콜 HTTP
* 2.1 HTTP는 클라이언트와 서버간에 통신을 한다
* 2.2 리퀘스트와 리스폰스를 교환하여 성립
* 2.3 HTTP는 상태를 유지하지 않는 프로토콜
* 2.4 리퀘스트 URI로 리소스를 식별
* 2.5 서버에 임무를 부여하는 HTTP 메소드
* 2.6 메소드를 사용해서 지시를 내리다
* 2.7 지속 연결로 접속량을 절약
  * 2.7.1 지속 연결
  * 2.7.2 파이프라인화
* 2.8 쿠키를 사용한 상태 관리

## 제 3장. HTTP 정보는 HTTP 메시지에 있다
* 3.1 HTTP 메시지
* 3.2 리퀘스트 메시지와 리스폰스 메시지의 구조
* 3.3 인코딩으로 전송 효율을 높이다
  * 3.3.1 메시지 바디와 엔티티 바디의 차이
  * 3.3.2 압축해서 보내는 콘텐츠 코딩
  * 3.3.3 분해해서 보내는 청크 전송 코딩
* 3.4 여러 데이터를 보내는 멀티 파트
* 3.5 일부분만 받는 레인지 리퀘스트
* 3.6 최적의 콘텐츠를 돌려주는 콘텐츠 네고시에이션

## 제 4장. 결과를 전달하는 HTTP 상태 코드
* 4.1 상태 코드는 서버로부터 리퀘스트 결과를 전달한다
* 4.2 2xx 성공(Success)
  * 4.2.1 200 OK
  * 4.2.2 204 No Content
  * 4.2.3 206 Partial Content
* 4.3 3xx 리다이렉트(Redirection)
  * 4.3.1 301 Moved Permanently
  * 4.3.2 302 Found
  * 4.3.3 303 See Other
  * 4.3.4 304 Not Modified
  * 4.3.5 307 Temporary Redirect
* 4.4 4xx 클라이언트 에러(Client Error)
  * 4.4.1 400 Bad Request
  * 4.4.2 401 Unauthorized
  * 4.4.3 403 Forbidden
* 4.5 5xx 서버 에러(Server Error)
  * 4.5.1 500 Internal Server Error
  * 4.5.2 503 Service Unavailable

## 제 5장. HTTP와 연계하는 웹 서버
* 5.1 1대로 멀티 도메인을 가능하게 하는 가상 호스트
* 5.2 통신을 중계하는 프로그램 : 프록시, 게이트웨이, 터널
  * 5.2.1 프록시
  * 5.2.2 게이트웨이
  * 5.2.3 터널
* 5.3 리소스를 보관하는 캐시
  * 5.3.1 캐시는 유효기간이 있다
  * 5.3.2 클라이언트 측에도 캐시가 있다

## 제 6장. HTTP 헤더
* 6.1 HTTP 메시지 헤더
* 6.2 HTTP 헤더 필드
  * 6.2.1 HTTP 헤더 필드는 중요한 정보를 전달한다
  * 6.2.2 HTTP 헤더 필드의 구조
  * 6.2.3 4종류의 HTTP 헤더 필드
  * 6.2.4 HTTP/1.1 헤더 필드 일람
  * 6.2.5 HTTP/1.1 이외의 헤더 필드
  * 6.2.6 End-to-end 헤더와 Hop-by-hop 헤더
* 6.3 HTTP/1.1 일반 헤더 필드
  * 6.3.1 Cache-Control
  * 6.3.2 Connection
  * 6.3.3 Date
  * 6.3.4 Pragma
  * 6.3.5 Trailer
  * 6.3.6 Transfer-Encoding
  * 6.3.7 Upgrade
  * 6.3.8 Via
  * 6.3.9 Warning
* 6.4 리퀘스트 헤더 필드
  * 6.4.1 Accept
  * 6.4.2 Accept-Charset
  * 6.4.3 Accept-Encoding
  * 6.4.4 Accept-Language
  * 6.4.5 Authorization
  * 6.4.6 Expect
  * 6.4.7 From
  * 6.4.8 Host
  * 6.4.9 If-Match
  * 6.4.10 If-Modified-Since
  * 6.4.11 If-None-Match
  * 6.4.12 If-Range
  * 6.4.13 If-Unmodified-Since
  * 6.4.14 Max-Forwards
  * 6.4.15 Proxy-Authorization
  * 6.4.16 Range
  * 6.4.17 Referer
  * 6.4.18 TE
  * 6.4.19 User-Agent
* 6.5 리스폰스 헤더 필드
  * 6.5.1 Accept-Ranges
  * 6.5.2 Age
  * 6.5.3 ETag
  * 6.5.4 Location
  * 6.5.5 Proxy-Authenticate
  * 6.5.6 Retry-After
  * 6.5.7 Server
  * 6.5.8 Vary
  * 6.5.9 WWW-Authenticate
* 6.6 엔티티 헤더 필드
  * 6.6.1 Allow
  * 6.6.2 Content-Encoding
  * 6.6.3 Content-Language
  * 6.6.4 Content-Length
  * 6.6.5 Content-Location
  * 6.6.6 Content-MD5
  * 6.6.7 Content-Range
  * 6.6.8 Content-Type
  * 6.6.9 Expires
  * 6.6.10 Last-Midified
* 6.7 쿠키를 위한 헤더 필드
  * 6.7.1 Set-Cookie
  * 6.7.2 Cookie
  * 6.8 그 이외의 헤더 필드
  * 6.8.1 X-frame-Option
  * 6.8.2 X-XSS-Protection
  * 6.8.3 DNT
  * 6.8.4 P3P

## 제 7장. 웹을 안전하게 하는 HTTPS
* 7.1 HTTP의 약점
  * 7.1.1 평문이기 때문에 도청 가능
  * 7.1.2 통신 상대를 확인하지 않기 때문에 위장 가능
  * 7.1.3 완전성을 증명할 수 없기 때문에 변조 가능
* 7.2 HTTP + 암호화 + 인증 + 완전성 보호 = HTTPS
  * 7.2.1 HTTP에 암호화와 인증과 완전성 보호를 더한 HTTPS
  * 7.2.2 HTTPS는 SSL의 껍질을 덮어쓴 HTTP
  * 7.2.3 상호간에 키를 교환하는 공개키 암호화 방식
  * 7.2.4 공개키가 정확한지 아닌지를 증명하는 증명서
  * 7.2.5 안전한 통신을 하는 HTTPS의 구조

## 제 8장. 누가 액세스하고 있는지를 확인하는 인증
* 8.1 인증이란?
* 8.2 BASIC 인증
  * 8.2.1 BASIC 인증 수순
* 8.3 DIGEST 인증
  * 8.3.1 DIGEST 인증 수순
* 8.4 SSL 클라이언트 인증
  * 8.4.1 SSL 클라이언트 인증의 인증 수순
  * 8.4.2 SSL 클라이언트 인증은 2-factor 인증에서 사용된다
  * 8.4.3 SSL 클라이언트 인증은 이용하는데 비용이 필요하다
* 8.5 폼 베이스 인증
  * 8.5.1 인증의 대부분은 폼 베이스 인증
  * 8.5.2 세션 관리와 쿠키에 의한 구현

## 제 9장. HTTP에 기능을 추가한 프로토콜
* 9.1 HTTP를 기본으로 하는 프로토콜
* 9.2 HTTP의 병목 현상을 해소하는 SPDY
  * 9.2.1 HTTP의 병목 현상
  * 9.2.2 SPDY 설계와 기능
  * 9.2.3 SPDY는 웹의 병목 현상을 해결하는가?
* 9.3 브라우저에서 양방향 통신을 하는 WebSocket
  * 9.3.1 WebSocket의 설계와 기능
  * 9.3.2 WebSocket 프로토콜
* 9.4 등장이 기다려지는 HTTP/2.0
* 9.5 웹 서버 상의 파일을 관리하는 WebDAV
  * 9.5.1 HTTP/1.1을 확장한 WebDAV
  * 9.5.2 WebDAV에서 추가된 메소드와 상태 코드

## 제 10장. 웹 콘텐츠에서 사용하는 기술
* 10.1 HTML
  * 10.1.1 대부분 웹 페이지는 HTML로 되어 있다
  * 10.1.2 HTML 버전
  * 10.1.3 디자인을 적용하는 CSS
* 10.2 다이나믹 HTML
  * 10.2.1 웹 페이지를 동적으로 변경하는 다이나믹 HTML
  * 10.2.2 HTML을 조작하기 쉽게 해주는 DOM
* 10.3 웹 애플리케이션
  * 10.3.1 웹을 사용해서 기능을 제공하는 웹 애플리케이션
  * 10.3.2 웹 서버와 프로그램을 연계하는 CGI
  * 10.3.3 Java에서 보급된 서블릿
* 10.4 데이터 송신에 이용되는 포맷이나 언어
  * 10.4.1 범용적으로 사용할 수 있는 마크업 언어 XML
  * 10.4.2 갱신 정보를 송신하는 RSS/Atom
  * 10.4.3 JavaScript에서 이용하기 쉽고 가벼운 JSON

## 제 11장. 웹 공격 기술
* 11.1 웹 공격 기술
  * 11.1.1 HTTP에는 보안 기능이 없다
  * 11.1.2 리퀘스트는 클라이언트 측에서 변조 가능
  * 11.1.3 웹 애플리케이션에 대한 공격 패턴
* 11.2 출력 값의 이스케이프 미비로 인한 취약성
  * 11.2.1 크로스 사이트 스크립팅
  * 11.2.2 SQL 인젝션
  * 11.2.3 OS 커맨드 인젝션
  * 11.2.4 HTTP 헤더 인젝션
  * 11.2.5 메일 헤더 인젝션
  * 11.2.6 디렉토리 접근 공격
  * 11.2.7 리모트 파일 인클루션
* 11.3 웹 서버의 설정이나 설계 미비로 인한 취약성
  * 11.3.1 강제 브라우징
  * 11.3.2 부적절한 에러 메시지 처리
  * 11.3.3 오픈 리다이렉트
* 11.4 세션 관리 미비로 인한 취약성
  * 11.4.1 세션 하이잭
  * 11.4.2 세션 픽세이션
  * 11.4.3 크로스 사이트 리퀘스트 포저리
* 11.5 기타
  * 11.5.1 패스워드 크래킹
  * 11.5.2 클릭 재킹
  * 11.5.3 DoS 공격
  * 11.5.4 백도어
