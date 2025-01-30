# \[객체지향 원칙] 명시적 의존성 원칙(Explicit Dependencies Principle)

**명시적 의존성 원칙**은 객체나 함수가 제대로 작동하는 데 필요한 요소를 **명확하게 요청**해야 한다는 원칙입니다. \
이 원칙은 두 가지 주요 개념으로 나눌 수 있습니다.

**1. 클래스 생성 시, 필요한 요소 명시하기**

* 클래스를 생성할 때, 그 클래스가 작동하는 데 필요한 모든 요소를 **명시적으로 인자로 받**아야 합니다.

**2. 함수 호출 시, 필요한 요소를 인자로 받기**

* 함수를 호출할 때, **전역 변수나 숨겨진 인프라 요소를 내부에서 몰래 사용**하지 말고, 필요한 모든 요소를 **인자로 전달받도록** 해야 합니다.

#### 왜 중요할까?

* **의존성 명시화**는 호출자가 함수나 클래스가 무엇을 필요로 하는지 명확히 알 수 있게 도와줍니다. 이를 통해 코드가 **더 투명**해지고, 호출자와의 **신뢰성**이 향상됩니다.
* **코드의 문서화**가 자연스럽게 이루어지며, **가독성**도 좋아집니다. 무엇을 필요로 하고 어떤 방식으로 작동하는지 알 수 있어, 코드 이해가 쉬워집니다.

## **명시적 의존성 원칙 위반 사례 # 1**

* **클래스 생성 시, 필요한 요소 명시하기로 해결하기**&#x20;

### 기존 코드&#x20;

```dart
class Rectangle {
  int _left = 0;
  int _top = 0;
  int _width = 0;
  int _height = 0;

  // Setter 메서드
  void setLeft(int val) => _left = val;
  void setTop(int val) => _top = val;
  void setWidth(int val) => _width = val;
  void setHeight(int val) => _height = val;

  void displayInfo() {
    print("Rectangle: left=$_left, top=$_top, width=$_width, height=$_height");
  }
}

void main() {
  final rect = Rectangle();
  rect.setLeft(0);
  rect.setTop(0);
  rect.setWidth(10);
  rect.setHeight(20);

  rect.displayInfo();
}
```

**문제점 정리**

1. **초기화 누락 가능성:**
   * 객체 생성 후 값을 일일이 설정해야 하므로 특정 속성이 초기화되지 않은 상태로 남을 위험이 있습니다.
   * 예: `SetWidth` 호출을 깜빡하면 `m_Width` 값이 초기화되지 않아 불완전한 객체가 됩니다.
2. **캡슐화 위반:**
   * `SetLeft`, `SetTop` 등의 setter 메서드를 통해 내부 상태가 외부에서 쉽게 수정될 수 있습니다.
   * 캡슐화 원칙에 따르면 내부 상태는 불변 또는 최소한 안전하게 관리되어야 합니다.
3. **사용 복잡성 증가:**
   * 객체를 사용할 때 필수 값 세팅이 누락되지 않도록 개발자가 직접 관리해야 합니다.
   * 코드 가독성이 떨어지고 유지보수 부담이 증가합니다.
4. **명시적 의존성 원칙 위반:**
   * 클래스가 제대로 작동하는데 필요한 정보를 명시적으로 요청하지 않고, 객체 생성 이후에 외부에서 설정을 요구합니다.
   * 호출자는 객체가 작동하는 데 필요한 모든 의존성을 파악하기 어려워집니다.

### 리팩토링된 코드 (완전한 생성자 준수)&#x20;

```dart
class Rectangle {
  final int left;
  final int top;
  final int width;
  final int height;

  // 모든 의존성을 명시적으로 요청하는 생성자
  Rectangle({
    required this.left,
    required this.top,
    required this.width,
    required this.height,
  });

  void displayInfo() {
    print("Rectangle: left=$left, top=$top, width=$width, height=$height");
  }
}

void main() {
  final rect = Rectangle(left: 0, top: 0, width: 10, height: 20);
  rect.displayInfo();
}
```

## **명시적 의존성 원칙 위반 사례 # 2**

* **함수 호출 시, 필요한 요소를 인자로 받아서 해결하기**&#x20;

### **기존 코드**

```dart
class Rectangle {

  void rotateAtViewCenter(double delta) {
    // View 중심점을 구합니다.
    final view = getOwnerView();
    final x = view.getCenterX();
    final y = view.getCenterY();

    // x, y에 맞춰 회전시킵니다.
    rotateAt(delta, x, y);
  }

  void rotateAt(double delta, double x, double y) {
    print("Rotating at ($x, $y) with delta $delta");
  }

  View getOwnerView() {
    return View();
  }
}

class View {
  double getCenterX() => 100.0;
  double getCenterY() => 50.0;
}

void main() {
  final rect = Rectangle();
  rect.rotateAtViewCenter(45.0);
}
```

***

#### **문제점 분석**

* `rotateAtViewCenter` 함수 내부에서 `getOwnerView()`를 통해 `View` 객체를 가져오고, `x`, `y` 값도 내부에서 계산됩니다.
* 이 방식은 **의존성을 숨기고** 있으므로, `rotateAtViewCenter` 함수 호출자는 어떤 의존성이 있는지 알지 못합니다.
* `rotateAtViewCenter` 함수가 `View` 객체에 강하게 결합되어 있어 **결합도**가 증가하고, **캡슐화**가 제대로 이루어지지 않습니다.

### **리팩토링 된 코드 (완전한 함수)**

```dart
class Rectangle {

  void rotateAtViewCenter(double delta, View view) {
    // View 중심점을 외부에서 구하고 명시적으로 전달
    final x = view.getCenterX();
    final y = view.getCenterY();

    // x, y에 맞춰 회전시킵니다.
    rotateAt(delta, x, y);
  }
  
  void rotateAt(double delta, double x, double y) {
    print("Rotating at ($x, $y) with delta $delta");
  }
}

class View {
  double getCenterX() => 100.0;
  double getCenterY() => 50.0;
}

void main() {
  final view = View();
  final rect = Rectangle();
  rect.rotateAtViewCenter(45.0, view);  // View 객체를 명시적으로 전달
}
```

#### **수정된 점**

* **`rotateAtViewCenter` 함수**는 이제 `Rectangle` 클래스의 멤버 함수로 포함되어 있으며, `Rectangle` 객체의 `rotateAt` 함수를 호출하여 회전합니다.
* `View` 객체는 외부에서 전달되며, 그 중심점(`x`, `y`) 값은 **`rotateAtViewCenter`** 함수 내에서 계산되어 **`rotateAt`** 함수에 전달됩니다.

#### **결과**

* `rotateAtViewCenter` 함수가 `Rectangle` 클래스 내에 있기 때문에, **회전 기능을 `Rectangle` 객체와 관련된 로직**으로 한정할 수 있으며, 코드 구조가 더 일관적이고 자연스럽습니다.
* `View` 객체를 전달받는 구조는 그대로 유지되어 **명시적 의존성**과 **유연성**을 제공합니다.

