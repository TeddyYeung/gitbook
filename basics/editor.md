---
icon: pen-to-square
---

# What type of files does the Android Runtime (ART) execute?

Android Runtime (ART)는 **DEX (Dalvik Executable)** 파일을 실행합니다.&#x20;

#### 1. **DEX 파일이란?**

DEX 파일은 **Dalvik Executable**의 약자로, Java 소스 코드를 컴파일한 후 생성되는 **바이트코드 파일**입니다.

* 안드로이드 애플리케이션은 보통 Java 또는 Kotlin으로 작성되고, 이를 컴파일하면 먼저 `.class` 파일이 생성됩니다.
* 그런 다음, 여러 `.class` 파일이 모여 하나의 DEX 파일로 변환됩니다.
  * 이 변환은 **dx 도구**나 최신 빌드 시스템에서 수행됩니다.

***

#### 2. **ART와 DEX 파일**

ART는 안드로이드의 애플리케이션 실행 환경으로, **DEX 파일**을 실행 가능한 형태로 처리합니다.

* ART는 **Just-In-Time (JIT)** 컴파일 또는 **Ahead-Of-Time (AOT)** 컴파일을 통해 DEX 바이트코드를 네이티브 코드로 변환하여 실행합니다.
  * **AOT 컴파일**: 앱 설치 시, DEX 파일을 네이티브 코드로 미리 변환하여 실행 속도를 높입니다.
  * **JIT 컴파일**: 실행 중인 애플리케이션의 DEX 파일 일부를 네이티브 코드로 즉시 변환합니다.

***

#### 3. **DEX 파일의 특징**

* **경량화**: DEX 파일은 모바일 환경에서 최적화된 경량 바이트코드 형식을 사용합니다.
* **멀티플랫폼 지원**: ART는 다양한 하드웨어 아키텍처에서 실행 가능하도록 설계되었습니다.
* **압축 및 최적화**: DEX 파일은 메모리와 성능을 최적화하기 위해 여러 `.class` 파일을 합치고 압축합니다.

***

#### 4. **관련된 파일들**

* **.odex 파일**: Optimized DEX 파일로, 실행 속도를 높이기 위해 DEX 파일이 최적화된 형태입니다.
* **APK 파일**: DEX 파일은 최종적으로 APK(애플리케이션 패키지) 안에 포함되어 배포됩니다.

