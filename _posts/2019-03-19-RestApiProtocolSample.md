---
layout: post
title:  "[Github] RestApiProtocolSample (POP + RxSwift + Alamofire + Codable)"
date:   2019-03-19 23:00:00 +0900
categories: [iOS, Swift, Github]
---

이번에는 제가 개인적으로 사용하기 위한 Network 모듈을 소개해보겠습니다.
최대한 POP하도록 구성해보았으니 참고하실분들은 참고하시고, 개선할 점이 있으시면 언제든 풀리퀘스트 부탁드립니다 :)

Github: [https://github.com/jmkim0213/RestApiProtocolSample](https://github.com/jmkim0213/RestApiProtocolSample)

----

### POP
- 프로토콜 지향적으로 설계하기 위해 RestApiProtocol을 구현했습니다.

### RxSwift
- 통신에 대한 결과 (성공, 실패 등)을 bind하여 처리하기위해 사용되었습니다.

### Alamofire
- 네트워크 통신을 직접적으로 하기 위해 사용한 라이브러리 입니다.

### Codable
- 마샬링을 위해 Swift에서 기본적으로 제공되는 Codable을 사용했습니다.

----

### 구성
1. protocol RestApiProtocol
- pop로 구현한 통신 프로토콜

2. enum RestResult
- 통신결과에 대한 열거형 (성공/실패)

3. enum RestMethod
- HttpMethod + Multipart 열거형

4. struct RestUploadFile
- 업로드할 파일에 대한 구조체

5. RestRunnable
- 마샬링을 Concurrent하게 처리하기 위한 유틸성 클로저

6. class RestApiQueue
- DataRequest 관리(취소)를 위해 구현한 객체

----
