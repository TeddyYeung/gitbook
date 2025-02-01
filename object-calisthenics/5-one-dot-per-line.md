# 규칙 5 : 한 줄에 한 점만 사용 (One Dot Per Line)

"한 줄에 한 점만 사용" 원칙은 디미터의 법칙과 깊은 연관이 있습니다.

**디미터의 법칙이란?**&#x20;

&#x20;"객체는 자신이 직접 생성한 객체, 자신에게 전달된 객체, 자신이 속한 클래스의 객체, 자신에게 의존하는 객체에게만 메서드를 호출해야 한다"는 원칙으로, 객체 간 결합도를 낮추고 시스템을 유연하고 유지보수하기 쉽게 만듭니다.

"한 줄에 한 점만 사용" 규칙은 디미터의 법칙을 구현하는 방법으로, 객체가 다른 객체의 내부 구조에 접근하는 것을 최소화하여 상호작용을 제한합니다.

#### Flutter에서 디미터 법칙 적용 예시

### **Before : 디미터 법칙 위반 코드**

```dart
class Customer {
  final Order order;

  Customer(this.order);

  String getProductName() {
    return order.product.name;  // 여러 점(.) 사용
  }
}

class Order {
  final Product product;

  Order(this.product);
}

class Product {
  final String name;

  Product(this.name);
}

void main() {
  final product = Product('Laptop');
  final order = Order(product);
  final customer = Customer(order);

  // 디미터 법칙 위반: 여러 점(.) 사용
  print(customer.order.product.name);  // 직접 객체의 내부 구조에 접근
}
```

위 코드에서는 `customer.order.product.name`처럼 **여러 점을 사용**하여 객체 간에 직접적인 내부 구조를 참조하고 있습니다. 이는 디미터 법칙을 위반하는 방식으로, 객체 간의 결합도가 높아지고 유지보수에 어려움을 겪을 수 있습니다.

### **After: 리팩토링 코드 (디미터 법칙 준수 코드)**

디미터 법칙을 준수하려면, **객체는 다른 객체의 내부 구조를 알지 않고 메서드를 통해 필요한 데이터를 요청**해야 합니다. \
리팩토링된 코드는 다음과 같습니다:

```dart
class Customer {
  final Order order;

  Customer(this.order);

  String getProductName() {
    return order.getProductName();  // Order 객체의 메서드를 호출
  }
}

class Order {
  final Product product;

  Order(this.product);

  String getProductName() {
    return product.name;  // Product 객체의 내부 구조를 숨기고 메서드를 통해 접근
  }
}

class Product {
  final String name;

  Product(this.name);
}

void main() {
  final product = Product('Laptop');
  final order = Order(product);
  final customer = Customer(order);

  // 디미터 법칙 준수: 한 줄에 한 점만 사용
  print(customer.getProductName());  // 메서드를 통해 정보를 요청
}
```

이렇게 리팩토링하면, `Customer` 객체는 `Order`나 `Product` 객체의 내부 구조를 알지 않고, `Order` 클래스의 `getProductName()` 메서드를 통해 필요한 정보를 간접적으로 얻습니다. 이렇게 하면 **한 줄에 한 점만 사용**하여 객체 간의 결합도를 줄일 수 있습니다.

#### 요약

* **디미터의 법칙**은 객체가 다른 객체의 내부 구조를 알지 않고, **메서드를 통해** 필요한 정보를 요청하도록 하는 원칙입니다.
* **"한 줄에 한 점만 사용"** 규칙은 디미터 법칙을 구현하는 좋은 방법으로, 객체 간 상호작용을 최소화하고 결합도를 낮춥니다.
* **리팩토링된 코드**는 객체들이 서로의 내부 구조를 알지 않고, **메서드 호출만을 통해** 정보를 얻도록 변경되었습니다.



### 참고&#x20;

#### - Problem: Law of Demeter Violated to Inappropriately make a Service Locator

{% embed url="https://github.com/TeddyYeung/guide-to-testable-code/blob/main/flaw-digging-into-colaborators.md" %}

