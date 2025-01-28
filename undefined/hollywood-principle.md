# \[객체지향 원칙] 헐리우드 원칙(Hollywood Principle)

#### 🌟 **헐리우드 원칙 (Hollywood Principle)란?**

헐리우드 원칙(Hollywood Principle)의 핵심은 다음 문장으로 요약됩니다:

> **"Don't call us, we'll call you." (우리를 호출하지 마세요. 우리가 호출하겠습니다.)**

**💡 설명**

헐리우드 원칙은 상위 수준의 객체가 하위 수준의 객체에게 직접 요청하는 것을 피하고, 대신 하위 객체가 필요할 때 상위 객체가 호출하는 구조입니다. 이는 제어의 흐름(Control Flow)을 역전(Inversion of Control, IoC)시켜 객체 간 결합도를 줄이는 데 도움이 됩니다.
