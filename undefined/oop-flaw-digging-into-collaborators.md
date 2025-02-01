---
description: >-
  Digging into Collaborators는 협력 객체의 내부 구조를 너무 깊게 파고드는 것을 의미합니다. 즉, 객체가 다른 객체와
  협력할 때, 그 객체의 내부 상태나 구현 세부사항에 지나치게 의존하는 경우입니다. 이런 접근은 디미터 법칙을 위반하는 방식으로 간주됩니다.
---

# \[OOP] Flaw: Digging into Collaborators

### 협력 객체(Collaborator)의 이해

* 협력 객체는 객체 지향 프로그래밍에서 특정 객체가 자신의 역할을 수행하기 위해 상호작용하는 다른 객체를 의미
* 객체는 모든 작업을 직접 처리하지 않고 협력 객체에 위임하여 책임을 분산시킵니다.

### 안티 패턴

**Digging into Collaborators**는 협력 객체의 내부에 과도하게 접근하는 안티패턴으로, 다음과 같은 문제를 야기합니다:

* 캡슐화 위반
* 객체 간 결합도 증가
* 단일 책임 원칙 위반

아래는 Flutter 코드 예제를 통해 "Digging into Collaborators" 문제를 설명하고 이를 개선하는 방법입니다.

### Digging into Collaborators 사례 1&#x20;

#### 🛑 문제 코드&#x20;

```dart
class Address {
  final String street; // Collaborator
  final String city; // Collaborator

  Address(this.street, this.city);
}

class Order {
  final Address address; // Collaborator

  Order(this.address);
}

class ShippingService {
  void shipOrder(Order order) {
    // ❌ 문제: Service가 Order 객체 내부로 파고들어 값 객체(Address)에 직접 접근함
    String destination = "${order.address.street}, ${order.address.city}";
    print("Shipping to: $destination");
  }
}

void main() {
  Order order = Order(Address("123 Main St", "Flutter City"));
  ShippingService().shipOrder(order);
}
```

#### ⚠️ 문제점 분석

* `ShippingService`가 `Order` 객체 내부의 `Address` 필드에 직접 접근해 데이터를 조합하는 방식은 캡슐화 원칙을 위반합니다.
* 이는 Order가 자신의 데이터를 관리하지 않고 외부 객체가 이를 조작하도록 만드는 결과를 초래합니다.

***

#### ✅ 개선된 코드 (Tell, Don't Ask 원칙 적용)

```dart
class Address {
  final String street;
  final String city;

  Address(this.street, this.city);

  String getFullAddress() {
    return "$street, $city";
  }
}

class Order {
  final Address address;

  Order(this.address);

  String getShippingAddress() {
    // Order 객체가 자신의 데이터를 활용해 필요한 정보를 제공
    return address.getFullAddress();
  }
}

class ShippingService {
  void shipOrder(Order order) {
    // ✅ 개선: Service가 Order에 요청하고 필요한 정보를 얻음
    String destination = order.getShippingAddress();
    print("Shipping to: $destination");
  }
}

void main() {
  Order order = Order(Address("123 Main St", "Flutter City"));
  ShippingService().shipOrder(order);
}
```

***

#### 🚀 개선 사항 요약

* `Order` 클래스가 자신에 대한 책임을 가지도록 `getShippingAddress` 메서드를 추가했습니다.
* 이제 `ShippingService`는 객체에 데이터를 요청하기만 하고 내부 구현에 관여하지 않습니다.

### Digging into Collaborators  사례 2

#### 🛑 Before: 테스트하기 어려운 설계

```dart
class TaxTable {
  double getTaxRate(Address address) {
    // 단순히 예제를 위해 주소에 따른 세율 반환
    return address.city == "Flutter City" ? 0.09 : 0.07;
  }
}

class Address {
  final String street; // Collaborator
  final String city; // Collaborator

  Address(this.street, this.city);
}

class User {
  final Address address; // Collaborator

  User(this.address);
}

class Invoice {
  final double subtotal; // Collaborator

  Invoice(this.subtotal);
}

class SalesTaxCalculator {
  final TaxTable taxTable; // Collaborator

  SalesTaxCalculator(this.taxTable);

  double computeSalesTax(User user, Invoice invoice) {
    // 문제점: 
    // - User 객체 내부의 Address ,Invoice 내부 subtotal에 직접 접근 (캡슐화 위반)
    // - SRP(Single Responsibility Principle) 위반: 계산 외에 데이터 추출 책임 포함
    Address address = user.address;
    double amount = invoice.subtotal;
    return amount * taxTable.getTaxRate(address);
  }
}
```

**테스트 코드 (Before)**

```dart
void main() {
  TaxTable taxTable = TaxTable();
  SalesTaxCalculator calc = SalesTaxCalculator(taxTable);

  Address address = Address("123 Main St", "Flutter City");
  User user = User(address);
  Invoice invoice = Invoice(95.00);

  double tax = calc.computeSalesTax(user, invoice);
  print("Tax: $tax"); // 테스트가 복잡하고 객체 그래프 초기화가 필요
}
```

#### 🚩 **테스트 코드 (Before) 문제점**

1. **객체 그래프 초기화 복잡성:**
   * `User`, `Address`, `Invoice` 객체를 모두 생성해야 하는 불필요한 초기화 과정이 테스트 코드에 포함되어 있어 복잡도가 증가합니다.
2. **강한 결합 (Tight Coupling):**
   * `SalesTaxCalculator`가 `User`와 `Invoice` 객체에 직접 의존하면서 불필요한 객체 참조가 발생합니다.
3. **테스트 유지보수 어려움:**
   * 요구사항이 변경되면 객체 생성 로직이 수정되어야 하므로 테스트 코드의 유지보수가 어려워집니다.
4. **SRP 위반:**
   * `SalesTaxCalculator`가 본래 책임인 세금 계산 외에 데이터 추출 로직도 수행하여 역할이 복잡해집니다.

***

#### ✅ After: 테스트하기 쉬운 설계

```dart
class SalesTaxCalculator {
  final TaxTable taxTable;

  SalesTaxCalculator(this.taxTable);
  
  // 위 다른  코드 생략
 
  double computeSalesTax(Address address, double amount) {
    return amount * taxTable.getTaxRate(address);
  }
}
```

**테스트 코드 (After)**

```dart
void main() {
  TaxTable taxTable = TaxTable();
  SalesTaxCalculator calc = SalesTaxCalculator(taxTable);

  Address address = Address("123 Main St", "Flutter City");

  // ✅ 단순한 테스트: 불필요한 객체 제거
  double tax = calc.computeSalesTax(address, 95.00);
  print("Tax: $tax"); // Tax: 8.55
}
```

***

#### ✨ 개선 요약

* **문제 해결:** SalesTaxCalculator가 더 이상 `User`, `Invoice` 같은 불필요한 객체에 의존하지 않고 필요한 값(`Address`, `double`)만 직접 받습니다.
* **테스트 코드 단순화:** 객체 그래프 초기화 작업을 제거하여 테스트가 단순해지고 명확해졌습니다.
* **유연성 증가:** 새로운 클래스 구조에도 쉽게 대응할 수 있는 테스트 가능한 설계입니다.

### **플러터 - Domain-Specific Language (DSL)의 경우에는 디미터 법칙 위반** 에서 예**외**

* DSL은 특정 도메인(예: 설정, 컴파일, 데이터 매핑 등)에 최적화된 언어입니다.
* 때로는 이러한 DSL이 디미터 법칙을 어기는 방식으로 설계될 수 있습니다.

**예시 코드**

```dart
AnimationController _controller = AnimationController(
  duration: const Duration(seconds: 2),
  vsync: this,
)..repeat()
 ..addListener(() => setState(() {}))
 ..forward();
```

#### **왜 디미터 법칙을 위반해도 괜찮은가?**

* `AnimationController`의 설정을 위해 `repeat()`, `addListener()`, `forward()`와 같은 메서드를 체이닝 방식으로 호출하고 있습니다.
* 디미터 법칙은 객체의 내부 속성에 직접 접근하지 않도록 권장하지만, 이 경우 **메서드 체이닝**을 사용하여 설정을 직관적이고 읽기 쉽게 만듭니다.
* 이 방식은 **값 객체**를 **쉽게 이해할 수 있도록 구성**하는 방식으로, 디미터 법칙을 위반해도 설정의 가독성과 직관성을 우선시하는 **허용 가능한 예외**입니다.

### 참고&#x20;

[https://github.com/mhevery/guide-to-testable-code/blob/main/flaw-digging-into-colaborators.md#problem-law-of-demeter-violated-to-inappropriately-make-a-service-locator](https://github.com/mhevery/guide-to-testable-code/blob/main/flaw-digging-into-colaborators.md#problem-law-of-demeter-violated-to-inappropriately-make-a-service-locator)
