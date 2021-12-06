---
layout: post
title: http vs https
categories: Network
---

# HTTP와 HTTPS

#### HTTP와 HTTPS는 뭐가 다를까?



## HTTP

우선 HTTP가 무엇인지 살펴봅시다. HTTP는 HyperText Transfer Protocol의 약자로, 서로 다른 시스템 사이에서 통신을 주고받게 해주는 프로토콜입니다. 80번 포트를 사용하며, 애플리케이션 레벨의 프로토콜로 TCP/IP 위에서 작동합니다. 또한, 상태를 가지고 있지 않는 Stateless 프로토콜입니다.

> 프로토콜 : 컴퓨터 간 메시지를 주고 받는 규칙

> Stateless : 서버에 클라이언트와의 세션 정보를 저장하지 않음

### HTTP의 구조

HTTP는 요청/응답(request/response) 구조로 되어있습니다. 클라이언트가 HTTP Request를 보내면 서버는 HTTP Response를 보내줍니다. HTTP 메시지는 start line, header, body로 구성됩니다.



#### HTTP Request

- start line : HTTP Request의 첫 라인입니다. HTTP Method, Request target, HTTP Version으로 구성됩니다.
  - HTTP Method : request가 의도한 동작을 정의합니다.
  - Request target : request가 전송되는 목표 URI입니다.
  - HTTP Version : 사용되는 HTTP 버전입니다.
- header : request에 대한 추가 정보를 담고 있는 부분입니다. Key : Value 구조로 되어있습니다.
- body : request의 실제 내용입니다. HTTP Method에 따라 body가 없는 경우도 있습니다.



#### HTTP Response

- start line : HTTP Version, Status Code, Status Text로 구성됩니다.
  - HTTP Version : 사용되는 HTTP 버전입니다.
  - [Status Code](https://ko.wikipedia.org/wiki/HTTP_%EC%83%81%ED%83%9C_%EC%BD%94%EB%93%9C) : 응답 상태를 나타내는 숫자로 된 코드입니다. 
  - Status Text : 응답 상태를 설명해 주는 부분입니다.
- header : HTTP Request의 header와 동일한 구조입니다. response에서만 사용하는 값들이 있습니다.
- body : HTTP Request의 body와 동일한 구조입니다.

---



## HTTP의 단점

1. 데이터를 평문으로 전송하기 때문에 도청당할 수 있습니다.
2. 통신 상대를 확인하지 않기 때문에 위장이 가능합니다.
3. 클라이언트, 서버가 보낸 정보를 누군가가 위조할 수 있습니다.



## HTTPS

HTTPS는 HTTP에 암호화 과정이 추가된 프로토콜로, 위에서 언급된 HTTP의 문제점을 해결했습니다. HTTPS는 SSL을 사용하며, 443번 포트를 사용합니다.



### SSL

SSL은 Secure Sockets Layer의 약자로, 암호화 기반 인터넷 보안 프로토콜입니다. 인터넷 통신의 정보 보호, 인증, 데이터 무결성을 보장합니다. 대칭키, 공개키 암호화를 모두 사용합니다.



### HTTPS 동작 과정

HTTPS는 Handshake -> 데이터 전송 -> 세션 종료의 과정으로 이루어집니다. 핸드 셰이킹 단계에서 SSL이 적용됩니다.

#### SSL Handshake

1. Client Hello

   - 클라이언트가 서버에 연락합니다. 클라이언트는 자신의 브라우저가 지원하는 암호화 방식(Cipher Suite)와 랜덤 데이터를 생성해 전송합니다.
   - 이미 SSL 핸드셰이킹 과정이 이루어졌다면 기존의 세션을 재활용하기 위해 세션 아이디를 전송합니다.

2. Server Hello

   - 서버가 클라이언트에 연락합니다. 서버는 클라이언트가 전송한 암호화 방식 중 하나를 선택해서 선택 된 암호화 방식, 서버 측 인증서, 랜덤 데이터를 생성해 전송합니다.

3. Certificate

   - 클라이언트의 브라우저는 신뢰할 수 있는 CA 목록을 가지고 있습니다. 클라이언트는 신뢰할 수 있는 CA에서 발급한 인증서인지 확인합니다. 

   > CA(Certificate Authority) : 다른 곳에서 사용하기 위한 디지털 인증서를 발급하는 기관

   - 클라이언트는 실제 CA 기관에서 발급한 인증서인지 확인하기 위해 인증서에 포함되어 있는 public key를 통해 복호화를 시도합니다. 복호화에 성공한다면 CA의 개인키로 암호화 된 문서임이 확인되기 때문에 서버를 신뢰할 수 있습니다.
   - 클라이언트는 1단계에서 전송한 랜덤 데이터와 2단계에서 받은 랜덤 데이터를 조합해 Pre Master Secret 값을 생성하고, 인증서에 포함되어 있는 서버의 공개키를 통해 암호화한 후 서버에 전송합니다.

4. Client Key Exchange

   - 서버는 Pre Master Secret 값을 자신의 개인키로 복호화합니다. 클라이언트와 서버는 Pre Master Secret 값을 공유하게 됩니다. 이 값으로 Session Key를 생성합니다.

5. Exchange Messages

- 클라이언트와 서버 간의 세션이 형성되었습니다. Session Key를 통해 암호화해서 데이터를 주고 받습니다.

