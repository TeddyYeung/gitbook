# 규칙 6 : 축약 금지 (Don't Abbreviate)

### 요약

1. **명확한 이름**: 역할이 드러나는 이름을 사용하라.
2. **축약 금지**: 의도 파악을 어렵게 하는 축약을 지양하라.
3. **단일 책임**: 메서드가 너무 길거나 복잡하다면 책임을 분리하라.

### 예시 1 : 메서드가 길어진다면?

메서드명이 지나치게 길다면, 해당 메서드가 **여러 책임을 동시에 수행하고 있는 것은 아닌지** 고민해 보자. \
메서드는 단일 책임 원칙(SRP)을 지켜야 하며, 이름은 짧더라도 그 기능이 명확해야 한다.

***

**안 좋은 예: 여러 작업을 처리하는 메서드**

```dart
String getCustomerDetails(Customer customer) {
  return 'Name: ${customer.getName()}, Address: ${customer.getAddress()}, Phone: ${customer.getPhoneNumber()}';
}
```

**좋은 예: 역할을 분리하고 명확한 이름 사용**

```dart
String formatCustomerName(Customer customer) => 'Name: ${customer.getName()}';
String formatCustomerAddress(Customer customer) => 'Address: ${customer.getAddress()}';
String formatCustomerPhone(Customer customer) => 'Phone: ${customer.getPhoneNumber()}';
```

***

### 예시 2: 변수명을 명확하게 쓰는 간단한 예시

**안 좋은 예: 축약된 변수명**

```dart
void calculate(List<int> nums) {
  int res = 0;
  for (var n in nums) {
    res += n;
  }
  print(res);
}
```

* `nums`, `res`, `n` 같은 이름은 **무엇을 의미하는지 명확하지 않음**.
* 코드의 의도를 파악하기 위해 더 많은 고민이 필요함.

***

**좋은 예: 명확한 변수명**

```dart
void calculateTotal(List<int> numbers) {
  int total = 0;
  for (var number in numbers) {
    total += number;
  }
  print(total);
}
```

* `numbers`, `total`, `number`는 이름만으로도 **역할과 의도를 바로 알 수 있음**.
* 읽기 쉽고, 유지보수도 용이함.

***

