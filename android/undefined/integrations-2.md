---
icon: plug-circle-plus
---

# Android Kotlin에서 Retrofit을 활용한 네트워크 통신

### 1. HTTP 클라이언트의 종류

안드로이드에서 사용할 수 있는 대표적인 HTTP 클라이언트는 아래와 같습니다:

<div align="left"><figure><img src="../../.gitbook/assets/image (15).png" alt="" width="312"><figcaption></figcaption></figure></div>

* **Ktor**: 코틀린 기반의 비동기 HTTP 클라이언트로, 서버 개발과 클라이언트 개발 모두에서 사용
* **OkHttp**: Retrofit의 기반이 되는 HTTP 클라이언트로, 간단한 HTTP 작업에 적합\
  \-  **커넥션 풀링, 캐싱, 인터셉터**와 같은 저수준 기능
* **Retrofit**: RESTful API 호출을 간편하게 처리할 수 있도록 설계된 고수준 HTTP 클라이언트\
  \- 실제 네트워크 요청을 처리하기 위해 내부적으로 OkHttp를 사용

### 2. Retrofit의 기본 개념

<div align="left"><figure><img src="../../.gitbook/assets/image (17).png" alt="" width="563"><figcaption></figcaption></figure></div>

Retrofit은 **인터페이스 기반**으로 작동하며, \
API 호출을 추상화하여 코드를 간단하고 가독성 좋게 작성할 수 있게 도와줍니다.

* `@HTTP_METHOD`: HTTP 요청 메서드를 나타냅니다. \
  예를 들어, `@GET`, `@POST`, `@PUT`, `@DELETE` 등이 있습니다.
* `suspend`: Kotlin Coroutines를 활용한 비동기 호출을 지원합니다.
* `Response<T>`: 서버에서 반환받을 데이터 타입을 지정합니다. `YourResponseModelClass`는 데이터 모델 클래스입니다.

\


\
\
