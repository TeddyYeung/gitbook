---
icon: image-landscape
---

# Gradle, Version Catalog 소개

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



### 버전 카탈로그란?

버전 카탈로그(Version Catalog)는 Gradle 7.0 이상에서 제공하는 기능으로, 프로젝트에서 사용하는 라이브러리, 플러그인, 버전 정보를 중앙에서 관리할 수 있게 해줍니다. 이 방식은 의존성 관리를 간소화하고 코드의 가독성과 유지보수성을 높이는 데 도움을 줍니다.

#### 버전 카탈로그의 구성 요소

1.  **\[versions]**:\
    \


    <div align="left"><figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt="" width="360"><figcaption></figcaption></figure></div>

    * 라이브러리와 플러그인에 사용되는 버전 정보를 정의합니다.
    * 중복되는 버전 정보를 하나의 장소에서 관리하여 유지보수를 간편하게 합니다.
2.  **\[libraries]**:\
    \


    <div align="left"><figure><img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" width="540"><figcaption></figcaption></figure></div>

    * 사용할 라이브러리와 그룹, 이름, 버전을 정의합니다.
    * `version.ref`를 사용하여 \[versions] 섹션에 정의된 버전을 참조합니다.
3.  **\[plugins]** (옵션):\


    <div align="left"><figure><img src="../.gitbook/assets/image (2).png" alt="" width="540"><figcaption></figcaption></figure></div>

    * 프로젝트에서 사용하는 Gradle 플러그인의 정보를 정의할 수 있습니다.

#### 버전 카탈로그 전체 예시

```
[versions]
coil = "2.2.0"
kotlin = "1.8.0"

[libraries]
coil = { group = "io.coil-kt", name = "coil", version.ref = "coil" }
kotlin-stdlib = { group = "org.jetbrains.kotlin", name = "kotlin-stdlib", version.ref = "kotlin" }

[plugins]
kotlin = { id = "org.jetbrains.kotlin.jvm", version = "1.8.0" }
```

#### 버전 카탈로그의 장점

1. **중앙화된 관리**:
   * 프로젝트 전반에 걸쳐 사용되는 라이브러리와 플러그인의 버전을 한 곳에서 관리할 수 있어 중복을 방지합니다.
2. **가독성 향상**:
   * 코드에서 직접 버전을 정의하지 않고 명명된 참조를 사용하므로 의존성 목록이 깔끔해집니다.
3. **자동 완성 지원**:
   * Android Studio와 같은 IDE에서 버전 카탈로그를 활용하면 의존성을 추가할 때 자동 완성과 구문 강조 기능을 사용할 수 있습니다.
4. **동기화 용이**:
   * 팀에서 작업할 때 버전이 일관되게 유지되므로 충돌 가능성을 줄일 수 있습니다.

***

### Gradle 설정에서의 사용 방법

버전 카탈로그를 사용하려면 `settings.gradle` 또는 `settings.gradle.kts` 파일에 아래 내용을 추가해야 합니다:

```
enableFeaturePreview('VERSION_CATALOGS')

dependencyResolutionManagement {
    versionCatalogs {
        libs {
            from(files("gradle/libs.versions.toml"))
        }
    }
}
```

그런 다음, `libs.versions.toml` 파일을 생성하고 위에서 소개한 구성 요소를 정의합니다.

#### 의존성 추가 예시

`build.gradle`에서 라이브러리를 추가할 때는 `libs` 객체를 사용합니다:

```
dependencies {
    implementation libs.coil
    implementation libs.kotlin.stdlib
}
```

***

버전 카탈로그는 대규모 프로젝트에서 특히 유용하며, 의존성 관리의 일관성과 효율성을 높이는 데 기여합니다. 이를 활용하면 유지보수와 협업의 부담을 줄이고, 코드의 품질을 향상시킬 수 있습니다.

***

### 결론

Gradle는 강력하면서도 유연한 빌드 자동화 도구로, 특히 Android 개발에서 필수적입니다. \
복잡한 애플리케이션 개발을 단순화하고, 라이브러리 관리와 빌드 커스터마이징을 통해 생산성을 높일 수 있습니다. Gradle의 주요 개념을 이해하고 활용하면 더 나은 개발 환경을 구축할 수 있습니다.
