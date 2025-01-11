---
icon: plug-circle-plus
---

# Android에서 인텐트를 활용한 앱 간 통신



**1. 인텐트의 종류**

Android에서 인텐트는 크게 두 가지로 나뉩니다:

1. **명시적 인텐트(Explicit Intent)**
   * 특정 애플리케이션이나 컴포넌트를 명확히 지정하여 실행합니다.
   * 주로 같은 애플리케이션 내에서 Activity, Service, 또는 BroadcastReceiver를 호출할 때 사용됩니다.
2. **암시적 인텐트(Implicit Intent)**
   * 수행할 작업(액션)을 선언하고, 이를 처리할 수 있는 애플리케이션을 Android 시스템이 선택하도록 합니다.
   * 예를 들어, 사용자가 이미지 공유를 요청하면 Android는 해당 작업을 처리할 수 있는 앱(예: 메신저, 이메일 앱 등)을 사용자에게 제시합니다.

***

**2. 인텐트 필터(Intent Filters)**

애플리케이션의 `AndroidManifest.xml` 파일에 정의된 **인텐트 필터**는 해당 애플리케이션이 처리할 수 있는 인텐트의 유형을 명시합니다.

*   **구조:**

    ```xml
    <activity android:name=".ExampleActivity">
        <intent-filter>
            <action android:name="android.intent.action.SEND" />
            <category android:name="android.intent.category.DEFAULT" />
            <data android:mimeType="text/plain" />
        </intent-filter>
    </activity>
    ```
* **작동 원리:**
  1. Android 시스템은 인텐트의 **액션(action)**, **카테고리(category)**, 및 \*\*데이터 타입(data)\*\*을 기준으로 해당 인텐트를 처리할 수 있는 애플리케이션을 검색합니다.
  2. 적합한 애플리케이션이 여러 개라면 사용자가 선택할 수 있도록 목록을 제공합니다.

***

**3. 인텐트 동작 및 사용**&#x20;

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption><p>인텐트 동작 과정 시각화</p></figcaption></figure>

1. **사용자 액션**
   * 메인 액티비티 클래스에서 사용자가 버튼을 클릭하면, `startActivity`를 호출하여 암시적 인텐트를 전달합니다.
2. **Android 시스템의 동작**
   * Android 프레임워크는 사용자의 기기에서 인텐트 필터를 검색하여 요청된 액션을 처리할 수 있는 앱을 찾습니다.
   * 일치하는 앱을 발견하면 해당 정보를 패키징하여 요청을 처리할 수 있는 액티비티의 `onCreate` 메서드로 전달합니다.
3. **여러 앱이 처리 가능한 경우**
   * 동일한 인텐트를 처리할 수 있는 애플리케이션이 여러 개라면, 사용자가 원하는 앱을 선택할 수 있도록 선택 창을 제공합니다.
4. **인텐트 활용**
   * 이렇듯 , 자신의 앱뿐만 아니라, 사용자의 기기 내 다른 앱들과도 흥미로운 상호작용을 구현할 수 있습니다.

**4. 코드 사용 예시** \


**1) 명시적 인텐트 (Explicit Intent)**\
\- 같은 앱 내에서 `Activity`를 호출:

```kotlin
val intent = Intent(this, SecondActivity::class.java)
intent.putExtra("message", "Hello, SecondActivity!")
startActivity(intent)
```

* `SecondActivity`가 명확히 지정되어 호출됩니다.
* 주로 내부 로직에서 화면 전환이나 데이터를 전달할 때 유용합니다.
  * 적용 사례:
    * 특정 화면으로 이동할 때: 로그인 완료 후 대시보드로 이동.
    * 서비스 시작: 음악 재생 서비스 시작.

**2) 암시적 인텐트 (Implicit Intent)**\
\- 텍스트 공유 기능 제공:

```kotlin
val shareIntent = Intent(Intent.ACTION_SEND)
shareIntent.type = "text/plain"
shareIntent.putExtra(Intent.EXTRA_TEXT, "이 텍스트를 공유합니다.")
startActivity(Intent.createChooser(shareIntent, "공유할 앱 선택"))
```

* 사용자는 해당 작업을 수행할 수 있는 앱(예: 메신저, 이메일 등)을 선택할 수 있습니다.
  * 적용 사례:
    * 이메일 전송 요청.
    * 브라우저를 통해 URL 열기.
    * 이미지 또는 텍스트 공유.

***

5. **인텐트 활용 시 유의사항**

* **보안:** 암시적 인텐트를 사용할 경우 민감한 데이터를 포함하지 않도록 주의해야 합니다.
* **호환성:** Android 버전에 따라 일부 인텐트 액션이 제한될 수 있으므로 최신 문서를 참고가  필요합니다.

***

이처럼 인텐트는 앱 간의 데이터 교환과 다양한 상호작용을 가능하게 하며, Android 개발에서 핵심적인 역할을 합니다.
