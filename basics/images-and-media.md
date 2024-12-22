---
icon: image-landscape
---

# Gradle 소개

Gradle는 Java, Kotlin, Android 프로젝트의 빌드를 자동화하고 관리하는 오픈 소스 도구입니다. Android Studio의 기본 빌드 시스템으로, 애플리케이션 개발 과정을 간소화하며 높은 유연성과 확장성을 제공합니다.

***

### Gradle란?

Gradle는 Apache Ant와 Maven의 장점을 결합한 빌드 시스템으로, Groovy 또는 Kotlin DSL로 작성된 스크립트를 사용해 빌드 과정을 자동화합니다.

#### 주요 특징

* **의존성 관리**: Maven Central, JCenter 등에서 라이브러리를 손쉽게 관리.
* **증분 빌드**: 변경된 파일만 처리해 빌드 시간을 단축.
* **멀티 프로젝트 지원**: 대규모 모듈 기반 프로젝트를 효율적으로 처리.
* **플러그인 생태계**: Android, Java 등 다양한 플러그인 제공.
* **크로스 플랫폼**: Windows, macOS, Linux에서 모두 사용 가능.

***

### Android 개발에서의 Gradle

Gradle는 Android 애플리케이션을 빌드, 테스트, 배포하는 데 최적화된 도구입니다. Android Studio와의 통합을 통해 더욱 편리하게 사용할 수 있습니다.

#### 핵심 개념

1.  **Build.gradle 파일**

    * Android 프로젝트는 보통 두 개의 `build.gradle` 파일을 포함합니다:
      * **프로젝트 수준**: 플러그인과 전역 설정 관리.
      * **모듈 수준**: 의존성과 빌드 타입 설정.

    예시:

    ```
    android {
        compileSdk 33

        defaultConfig {
            applicationId "com.example.myapp"
            minSdk 21
            targetSdk 33
            versionCode 1
            versionName "1.0"
        }
    }

    dependencies {
        implementation "androidx.core:core-ktx:1.10.1"
        implementation "com.google.android.material:material:1.9.0"
    }
    ```
2. **Gradle Wrapper**
   * 프로젝트 내 `gradlew` 스크립트를 사용해 모든 환경에서 동일한 Gradle 버전을 보장.
3. **Task와 Project**
   * Gradle은 작업(Task)과 프로젝트(Project) 개념을 기반으로 동작.
   *   예시:

       ```
       ./gradlew build  # 빌드 실행
       ./gradlew clean  # 빌드 파일 삭제
       ./gradlew test   # 테스트 실행
       ```

***

### Gradle의 강점

* **유연한 설정**: 빌드 과정 및 결과를 커스터마이징 가능.
* **효율성**: 증분 빌드와 캐시 활용으로 빠른 속도 제공.
* **확장성**: 대규모 모듈 프로젝트에 적합.
* **라이브러리 관리**: 버전 카탈로그로 의존성 관리 간소화.

#### 버전 카탈로그 예시

```
[versions]
coil = "2.2.0"

[libraries]
coil = { group = "io.coil-kt", name = "coil", version.ref = "coil" }
```

***

### 결론

Gradle는 강력하면서도 유연한 빌드 자동화 도구로, 특히 Android 개발에서 필수적입니다. \
복잡한 애플리케이션 개발을 단순화하고, 라이브러리 관리와 빌드 커스터마이징을 통해 생산성을 높일 수 있습니다. Gradle의 주요 개념을 이해하고 활용하면 더 나은 개발 환경을 구축할 수 있습니다.
