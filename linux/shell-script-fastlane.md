# 플러터에서 Shell Script로 Fastlane 두 개를 병렬 실행하는 방법

####

```makefile
space_auth:
    @echo "Running spaceAuth..."
    @yes y | fastlane spaceauth -u {yourEmail}
```

* `spaceauth`는 2FA 토큰 기반 인증인 Fastlane의 기능으로, \
  App Store Connect API 인증을 위해 사용자에게 임시 인증 토큰을 생성합니다.
* 예전에 사용해봤던 App Store Connect API Key를 발급받아 사용하는 인증 방식이 관리하기 더 편리하지만, \
  사용 간편함을 위해 `spaceauth를 사용중입니다.`&#x20;
* `fastlane spaceauth -u {yourEmail}`: 해당 이메일로 Apple 계정에 로그인하여 인증을 진행합니다.
* `yes y |`: 인증 과정에서 요구되는 "확인" 응답을 자동으로 처리합니다.

```makefile
deploy:
    @$(MAKE) space_auth
    @sh deploy.sh
```

* **역할**:
  * `space_auth`를 먼저 실행하여 인증을 갱신한 후, `deploy.sh` 스크립트를 실행하여 실제 배포 작업을 수행합니다.
  * 이를 통해 2FA 인증이 만료되었거나 새로운 토큰이 필요한 경우에도 문제없이 배포가 가능합니다.



**1. Fastlane 병렬 실행의 핵심**

* Fastlane은 기본적으로 단일 프로세스(싱글 쓰레드)로 동작합니다. \
  이를 통해 안정성을 유지할 수 있지만, 병렬 처리가 기본적으로 지원되지 않기 때문에 \
  AOS(Android)와 iOS를 동시에 배포하는 병렬 배포는 불가능한 한계가 있습니다.
* Shell Script를 이용해 각각의 Fastlane 프로세스를 독립적으로 실행하면 병렬 배포가 가능합니다.

***

**2. Shell Script로 병렬 실행하기**

```bash
#!/bin/bash

if [[ "$platform" == "$platform_ios" ]]; then
    # iOS 배포 함수 실행
    deploy_ios
elif [[ "$platform" == "$platform_aos" ]]; then
    # AOS(Android) 배포 함수 실행
    deploy_aos
elif [[ "$platform" == "$platform_both" ]]; then
    # 기존 순차 배포: AOS 배포 후 iOS 배포 실행 (동기적 처리)
    # deploy_aos && deploy_ios

    # 아래는 병렬 배포 (비동기 처리)
    deploy_aos &              # AOS 배포를 백그라운드에서 실행
    pid_aos=$!                # 백그라운드에서 실행된 AOS 배포 프로세스의 PID 저장

    deploy_ios &              # iOS 배포를 백그라운드에서 실행
    pid_ios=$!                # 백그라운드에서 실행된 iOS 배포 프로세스의 PID 저장

    # AOS 배포 프로세스의 종료를 기다리고 종료 상태를 확인
    wait $pid_aos             # AOS 배포 프로세스가 종료될 때까지 대기
    status_aos=$?             # AOS 배포 프로세스의 종료 상태를 저장 (0: 성공, 그 외: 실패)

    # iOS 배포 프로세스의 종료를 기다리고 종료 상태를 확인
    wait $pid_ios             # iOS 배포 프로세스가 종료될 때까지 대기
    status_ios=$?             # iOS 배포 프로세스의 종료 상태를 저장 (0: 성공, 그 외: 실패)

    # 각 배포 결과 출력
    echo "AOS deploy exit status: $status_aos"
    echo "iOS deploy exit status: $status_ios"

    # 두 배포 작업이 모두 성공했는지 확인
    if [[ $status_aos -eq 0 && $status_ios -eq 0 ]]; then
      # AOS와 iOS 배포 모두 성공한 경우
      echo "✅ Both AOS and iOS deployed successfully."
      return 0
    else
      # 하나 이상의 배포 작업이 실패한 경우
      echo "❌ One or both deployments failed."
      return 1
    fi
fi

```

* **`&`**: 명령어를 백그라운드에서 실행.
* **`wait`**: 백그라운드 작업이 끝날 때까지 대기.
* **`wait`** $pid\_aos , wait $pid\_ios  명령이 실행된 후, `$?` 변수에는 **`pid_ios` 프로세스의 종료 상태**가 저장

**실행 과정**

```bash
wait $pid_ios      # pid_ios 프로세스가 종료될 때까지 대기
status_ios=$?      # $? 변수에는 pid_ios 프로세스의 종료 상태가 저장됩니다.
```

*   여기서 `$?`의 값은 `pid_ios` 프로세스가 종료되었을 때 반환한 값입니다.

    * **0**: iOS 배포(`deploy_ios`)가 정상적으로 완료됨.
    * **1 이상**: 배포 과정에서 에러가 발생함.



    #### **`$?` 값의 해석**

    **값의 종류**

    * **0**: 성공적으로 완료됨.
    * **1\~255**: 실패한 원인을 나타냄.
      * 각 값은 보통 특정 에러를 의미하며, 프로그램/명령어마다 다릅니다.
        * 예:
          * `1`: 일반적인 실행 실패.
          * `2`: 잘못된 명령 인수.
          * `127`: 명령어를 찾을 수 없음(파일 또는 디렉토리가 없음).



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

### **PID (Process ID)란?**

**PID란?**

* \*\*PID (Process ID)\*\*는 운영 체제에서 **각 프로세스를 고유하게 식별하는 숫자**입니다.
* 새로운 프로세스가 생성될 때, 운영 체제가 해당 프로세스에 PID를 할당합니다.

**PID의 특징**

1. **고유성**
   * 시스템에서 동시에 실행 중인 프로세스는 고유한 PID를 가집니다.
   * 단, 프로세스가 종료되면 해당 PID는 재사용될 수 있습니다.
2. **범위**
   * PID는 운영 체제에 따라 범위가 다릅니다. 일반적으로 1에서 시작하며, 최대값은 시스템 설정에 따라 달라집니다.
     * 예: Linux의 기본 PID 범위는 1\~32768.
3. **부모-자식 관계**
   * 모든 프로세스는 부모 프로세스를 가지며, 부모의 PID를 통해 관계를 추적할 수 있습니다.
   * 예: 스크립트에서 실행한 작업(`deploy_aos`)은 스크립트 프로세스의 자식 프로세스가 됩니다.

**PID의 역할**

* PID는 프로세스를 제어하거나 모니터링할 때 사용됩니다.
  * 특정 프로세스를 **종료**: `kill [PID]`
  * 프로세스 상태 확인: `ps -p [PID]`
  * 대기(wait) 처리: `wait [PID]`



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
