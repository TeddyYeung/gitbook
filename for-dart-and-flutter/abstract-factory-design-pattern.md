# Abstract Factory Design Pattern

**개요**

Abstract Factory 패턴은 관련 객체들의 제품군(Product Family)을 생성하기 위한 인터페이스를 정의하고, \
이를 구현하는 구체적인 공장(Factory) 클래스를 제공하는 생성 패턴입니다.\
객체 생성 로직과 클라이언트 코드가 분리되어, 코드의 확장성과 유지보수성을 향상시킬 수 있습니다.

***

**사용 목적**

* **플랫폼별 객체 생성:** Android와 iOS처럼 환경에 따라 다른 객체 생성이 필요할 때
* **UI 테마 지원:** 다크 모드와 라이트 모드에 따른 위젯 구성을 분리하고자 할 때
* **제품군 관리:** 연관된 객체들이 일관되게 생성되도록 보장할 때
* **딥링크 이벤트 분기 처리:**\
  딥링크 유형에 따라 **마케팅 캠페인**, **알림톡**, **일반 딥링크** 등 서로 다른 이벤트 처리가 필요한 경우, \
  Factory 클래스를 통해 각 이벤트 로직을 캡슐화하여 관리할 수 있음

***

**구성 요소**

* **Abstract Factory:** 객체 생성 메서드를 정의하는 인터페이스
* **Concrete Factory:** 실제 객체 생성을 담당하는 클래스
* **Abstract Product:** 생성될 객체의 공통 인터페이스
* **Concrete Product:** 구체적으로 생성된 객체

***

#### **Flutter 예제**

**1. Abstract Product 정의**

```dart
dart복사편집abstract class Button {
  void render();
}

abstract class Checkbox {
  void render();
}
```

**2. Concrete Product 구현**

```dart
dart복사편집class AndroidButton implements Button {
  @override
  void render() => print("Android Button");
}

class IOSButton implements Button {
  @override
  void render() => print("iOS Button");
}

class AndroidCheckbox implements Checkbox {
  @override
  void render() => print("Android Checkbox");
}

class IOSCheckbox implements Checkbox {
  @override
  void render() => print("iOS Checkbox");
}
```

**3. Abstract Factory 정의**

```dart
dart복사편집abstract class WidgetFactory {
  Button createButton();
  Checkbox createCheckbox();
}
```

**4. Concrete Factory 구현**

```dart
dart복사편집class AndroidWidgetFactory implements WidgetFactory {
  @override
  Button createButton() => AndroidButton();

  @override
  Checkbox createCheckbox() => AndroidCheckbox();
}

class IOSWidgetFactory implements WidgetFactory {
  @override
  Button createButton() => IOSButton();

  @override
  Checkbox createCheckbox() => IOSCheckbox();
}
```

**5. 클라이언트 코드**

```dart
dart복사편집void renderUI(WidgetFactory factory) {
  final button = factory.createButton();
  final checkbox = factory.createCheckbox();

  button.render();
  checkbox.render();
}

void main() {
  // Android UI 렌더링
  WidgetFactory factory = AndroidWidgetFactory();
  renderUI(factory);

  // iOS UI 렌더링
  factory = IOSWidgetFactory();
  renderUI(factory);
}
```

***

**적용 사례**

* **Flutter 플랫폼 위젯 관리:** Material과 Cupertino 위젯을 조건에 따라 분리
* **테마 관리:** 다크 모드와 라이트 모드에 따른 위젯 그룹 생성
* **다국어 지원:** 언어에 따라 다른 UI 텍스트 또는 컴포넌트 생성
* **딥링크 이벤트 대응:** 딥링크 유형에 따른 이벤트 분기 처리
  * **마케팅 딥링크:** 이벤트 페이지 또는 프로모션 화면으로 연결
  * **알림톡 딥링크:** 푸시 메시지 또는 알림에 따른 특정 화면 이동
  * **일반 딥링크:** 메인 화면 또는 기본 화면으로 연결

Abstract Factory 패턴을 적용하면 특정 플랫폼이나 테마 변화에 따라 코드 수정을 최소화하면서 일관성 있는 구조를 유지할 수 있습니다.
