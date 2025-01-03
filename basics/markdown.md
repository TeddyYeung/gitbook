---
icon: markdown
---

# Android SDK와 주요 도구들

Android Software Development Kit(SDK)는 Android 애플리케이션을 개발, 테스트, 디버깅하기 위한 필수 도구와 라이브러리입니다. 여기서는 SDK의 주요 구성 요소와 역할을 간단히 정리했습니다.

***

### 1. **Android SDK**

Android SDK는 애플리케이션 개발에 필요한 라이브러리, API, 에뮬레이터, 유틸리티를 제공합니다. Android Studio를 통해 다운로드하고 관리할 수 있습니다.

***

### 2. **SDK 도구**

SDK 도구는 디버깅, 빌드 생성, 에뮬레이터 관리 등 다양한 개발 작업을 돕는 명령줄 유틸리티입니다.

#### **SDK Build Tools**

* 앱의 소스 코드를 APK(Android Package)로 컴파일합니다.
* `aapt`, `zipalign` 같은 도구로 리소스를 최적화하고 패키징합니다.

#### **SDK Platform Tools**

* Android 기기와 상호작용할 수 있는 도구 모음입니다.
* 주요 도구: **adb(Android Debug Bridge)**, **fastboot**
* 최신 Android 플랫폼과 호환성을 유지합니다.

#### **Android Emulator**

* 실제 Android 기기를 가상으로 구현합니다.
* 다양한 Android 버전, 화면 크기, 하드웨어 설정에서 앱을 테스트할 수 있습니다.
* 스냅샷 저장, 네트워크 조건 시뮬레이션 등의 고급 기능을 제공합니다.

***

### 3. **Android Debug Bridge(adb)**

**adb**는 디버깅 및 기기 관리에 유용한 명령줄 도구입니다. 주요 기능은 다음과 같습니다.

#### **에뮬레이터 및 기기 상태 관리**

*   `adb` 명령으로 에뮬레이터 또는 기기를 시작, 중지, 재설정할 수 있습니다.

    예시:

    ```
    adb devices
    ```

    연결된 기기 목록과 상태를 확인합니다.

#### **APK 설치**

*   앱을 기기에 직접 설치하여 테스트합니다.

    예시:

    ```
    adb install <apk 경로>
    ```

#### **파일 전송**

*   개발 환경과 기기 간에 파일(로그, 미디어 등)을 주고받을 수 있습니다.

    예시:

    ```
    adb push <로컬 경로> <기기 경로>
    adb pull <기기 경로> <로컬 경로>
    ```

***

### 4. **주요 활용 사례**

* **디버깅**: `adb logcat`으로 실시간 로그를 확인하고 문제를 진단합니다.
* **다양한 환경 테스트**: 서로 다른 Android 버전과 하드웨어 사양의 에뮬레이터를 설정합니다.
* **자동화**: `adb` 명령을 활용해 배포, 로그 추출 같은 반복 작업을 간소화합니다.

***

### 결론

`adb` 같은 도구를 활용하면 디버깅, 테스트, 배포 과정을 효율적으로 수행할 수 있습니다.&#x20;
