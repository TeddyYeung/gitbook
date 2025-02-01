# \[객체지향 원칙] 묻지 말고 말하라 TDA 원칙(Tell, Don’t Ask Principle)

클린코드 6장, 객체와 자료 구조

객체지향 프로그래밍에서 "묻지 말고 말하라" 원칙

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

**정의**

* 객체의 내부 상태를 직접 묻지 않고, 필요한 작업을 객체에게 요청하는 설계 원칙.

**효과**

* 캡슐화와 응집력 강화
* 코드의 확장성과 유지보수성 향상

**핵심 내용**

1. 객체의 정보를 Getter로 가져오지 말 것.
2. 객체 외부에서 로직을 구현하지 말 것.
3. 객체 내부에 로직을 구현하여 정보 은닉을 유지할 것.

**설명**

* **잘못된 예시**: 객체의 정보를 외부에서 직접 획득하여 처리
* **올바른 예시**: 객체가 스스로 처리하도록 요청

**목표**

* 객체가 자신의 책임을 수행하도록 설계
* 외부와의 결합도를 낮춰 유연한 코드 구조 유지

&#x20;

이번 글에서는 "묻지 말고 말하라" 원칙의 위반 사례와 이를 준수하는 방법을 **Flutter**를 사용하여 설명하겠습니다.

***

### 위반 사례 <a href="#undefined" id="undefined"></a>

Rectangle 클래스에서 중심점을 구하는 예제를 살펴보겠습니다. 아래는 잘못된 설계의 Dart 코드입니다:

```dart
class Rectangle {
  final int left;
  final int top;
  final int width;
  final int height;

  Rectangle({required this.left, required this.top, required this.width, required this.height});
}

void main() {
  Rectangle rect = Rectangle(left: 0, top: 0, width: 10, height: 20);

  // 중심점을 계산하기 위해 내부 정보를 묻는 코드
  int centerX = rect.left + rect.width ~/ 2;
  int centerY = rect.top + rect.height ~/ 2;

  print('Center: ($centerX, $centerY)');
}
```

위 코드에서는 `left`, `top`, `width`, `height`와 같은 내부 정보를 여러 번 가져와서 중심점을 계산하고 있습니다.\
이는 **캡슐화**를 약화시키며, "묻지 말고 말하라" 원칙을 위반한 사례입니다.

***

### 준수 방법 <a href="#undefined" id="undefined"></a>

Rectangle 클래스에 중심점을 계산하는 메서드를 추가하면 이 원칙을 준수할 수 있습니다. 개체의 내부 상태를 묻지 않고, 작업을 요청하는 방식으로 변경해봅시다:

```dart
class Rectangle {
  final int left;
  final int top;
  final int width;
  final int height;

  Rectangle({required this.left, required this.top, required this.width, required this.height});

  /// 중심점을 계산하는 메서드
  Offset getCenter() {
    return Offset(left + width / 2, top + height / 2);
  }
}

void main() {
  Rectangle rect = Rectangle(left: 0, top: 0, width: 10, height: 20);

  // 중심점을 요청하는 코드
  Offset center = rect.getCenter();
  print('Center: (${center.dx}, ${center.dy})');
}
```

#### 코드 설명 <a href="#undefined" id="undefined"></a>

* `getCenter()` 메서드를 통해 중심점을 계산하는 책임을 `Rectangle` 클래스에 부여했습니다.
* `Offset` 객체를 사용하여 좌표를 표현하며, `center.dx`와 `center.dy`로 접근할 수 있습니다.
* 이제 `Rectangle` 클래스의 내부 구현을 수정하더라도, `getCenter()`를 사용하는 코드에는 영향을 미치지 않습니다.

***

### 은닉성 강화: 내부 상태 변경 <a href="#undefined" id="undefined"></a>

다음은 `Rectangle` 클래스의 내부 상태를 중심점으로 표현하도록 변경한 예제입니다:

```dart
class Rectangle {
  final double centerX;
  final double centerY;
  final int width;
  final int height;

  Rectangle({required int left, required int top, required this.width, required this.height})
      : centerX = left + width / 2,
        centerY = top + height / 2;

  /// 중심점을 반환하는 메서드
  Offset getCenter() {
    return Offset(centerX, centerY);
  }
}

void main() {
  Rectangle rect = Rectangle(left: 0, top: 0, width: 10, height: 20);

  Offset center = rect.getCenter();
  print('Center: (${center.dx}, ${center.dy})');
}
```

#### 코드 설명 <a href="#id-1" id="id-1"></a>

* 내부 상태를 `centerX`와 `centerY`로 변경하여 중심점을 직접 저장하도록 설계했습니다.
* 생성자에서 `left`와 `top` 값을 기반으로 중심점을 계산하여 저장합니다.
* 이는 "묻지 말고 말하라" 원칙을 더욱 강력히 준수하며, 캡슐화와 응집력을 크게 향상시킵니다.

**참고**

[![](https://randomthoughtsonjavaprogramming.blogspot.com/favicon.ico)Trainwreck vs. Method Chaining](https://randomthoughtsonjavaprogramming.blogspot.com/2013/10/trainwreck-vs-method-chaining.html)

[#9. \[개체지향 원칙\] 묻지 말고 말하라 원칙(Tell, Don’t Ask Principle)](https://tango1202.github.io/principle/principle-tell-dont-ask/)
