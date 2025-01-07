# 플러터에서 Shell Script로 Fastlane 두 개를 병렬 실행하는 방법

#### **플러터에서 Shell Script로 Fastlane 두 개를 병렬 실행하는 방법**

**1. Fastlane 병렬 실행의 핵심**

Fastlane은 기본적으로 \*\*단일 프로세스(싱글 쓰레드)\*\*로 동작합니다. Shell Script를 이용해 각각의 Fastlane 프로세스를 독립적으로 실행하면 병렬 배포가 가능합니다.

***

**2. Shell Script로 병렬 실행하기**

```bash
#!/bin/bash

# iOS와 Android Fastlane 명령어 병렬 실행
fastlane ios beta &    # iOS 배포를 백그라운드에서 실행
fastlane android beta  # Android 배포 실행

# 모든 프로세스가 종료될 때까지 대기
wait
echo "iOS and Android Fastlane processes completed."
```

* **`&`**: 명령어를 백그라운드에서 실행.
* **`wait`**: 백그라운드 작업이 끝날 때까지 대기.

***

**3. Fastlane이 병렬 실행을 지원하지 않는 이유**

1. **상태 관리**:
   * 환경 변수, 로그, 인증 토큰 등의 공유 상태로 인해 병렬 실행 시 충돌 위험.
2. **리소스 잠금**:
   * 동일한 프로젝트 디렉토리에서 빌드 결과물이나 파일 잠금으로 충돌 가능.
3. **순차 실행 설계**:
   * 안정성과 디버깅을 위해 워크플로우를 순차적으로 실행하도록 설계.

***

**4. 병렬 실행의 원리**

* Shell Script는 **운영 체제의 프로세스 관리** 기능을 이용해 독립적으로 Fastlane 프로세스를 실행.
* Fastlane 자체의 병렬 처리 제한을 우회하며, 각 프로세스는 운영 체제가 관리.

***

**5. 병렬 실행 시 주의 사항**

1. **리소스 관리**:
   * iOS와 Android 빌드 디렉토리 및 캐시가 충돌하지 않도록 설정.
2.  **로그 관리**:

    * 병렬 실행 시 로그 출력이 섞일 수 있으므로 별도의 파일로 저장 추천.

    ```bash
    fastlane ios beta > ios_log.txt 2>&1 &
    fastlane android beta > android_log.txt 2>&1
    ```
3. **환경 변수 분리**:
   * Fastlane 실행 시 사용하는 환경 변수를 명시적으로 분리.

***

**결과**\
\
**Build clean 후 배포 테스트 결과**

`flutter clean`을 적용하고, 아래의 명령어로 각 플랫폼의 빌드 디렉토리를 정리한 후 배포 순서를 변경하여 테스트한 결과는 다음과 같습니다:

```bash
flutter clean
rm -rf android/build
rm -rf ios/Pods ios/Flutter/Flutter.framework ios/build
rm -rf build/
```

1. **첫 번째 테스트 (병렬 배포 먼저)**:
   * 병렬 배포: ⏱️ **260초** (약 4\~5분)
   * 순차 배포: **328초** (약 5\~6분)
2. **두 번째 테스트 (순차 배포 먼저)**:
   * 순차 배포: ⏱️ **331초** (약 5\~6분)
   * 병렬 배포: **249초** (약 4\~5분)

위 결과에서 병렬 배포가 항상 **순차 배포**보다 빠르게 완료되었습니다.\


**결론**

Fastlane 병렬 실행은 Shell Script를 이용한 **독립적인 프로세스 관리**로 구현할 수 있습니다. iOS와 Android 배포를 동시에 처리할 수 있어 빌드 및 배포 시간을 단축시킬 수 있습니다.
