---
icon: plug-circle-plus
---

# 람다 - 함수형 인터페이스(SAM 인터페이스)

#### #SAM 인터페이스

**1) SAM이란?**&#x20;

* Single abstract method / 단일 추상 메소드&#x20;

**2) SAM 인터페이스란?**

* 추상 메소드가 단 1개 있는 인터페이스
* '함수형 인터페이스' 라고도 한다(Functional interface)

**3) 대표적인 SAM 인터페이스는?**&#x20;

* View.java의 OnclickListener

```
public interface OnClickListener {
    /**
     * Called when a view has been clicked.
     *
     * @param v The view that was clicked.
     */
    void onClick(View v);
}
```

&#x20;

**4) 갑자기 람다와 SAM 인터페이스라니... 무슨 상관인가?**

* Sam인터페이스를 인자로 받는  Java함수를 호출 할 경우, Sam 인터페이스 객체 대신 람다를 넘길 수 있다는 사실
* 대표적으로 'Sam인터페이스를 인자로 받는  Java 메소드'는 View.setOnClickListener를 꼽을 수 있다

```
public void setOnClickListener(@Nullable OnClickListener l) {
             ...
}
```

&#x20;

\[Java에서 기본적인 setOnClickListener]는 익명 객체를 생성해서 전달 해야 하는 것이 기본인데

```
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {//익명객체 생성 전달}
});
```

&#x20;

이 경우 Kotlin에서, Sam 인터페이스 객체(익명 객체) 대신 람다를 전달 할 수 있다는 뜻

```
button.setOnClickListener { println("익명객체 대신 람다를 넘김 ${it.id} ") }
```

&#x20;

하지만 사실 Java에서도 버전 8이상에서는 동일하게 람다를 사용할 수 있었음

```
button.setOnClickListener(v -> Log.i("syTest","Java 에서의 람다"));
```

&#x20;

즉 코틀린에서도 Sam인터페이스(함수형 인터페이스)를 인자로 취하는 자바 메소드를 호출할 때 람다를 넘길 수 있도록 해준다는 뜻으로, Java와 호환성을 지원한다는 뜻

&#x20;

**어떻게 이런 동작이 가능한가?**

* 람다를 함수형 인터페이스의 인자로 전달 할 때, 내부적으로 컴파일러가, 람다에 대해 익명 객체를 생성해서 메소드에 넘김
* 익명 객체를 직접 생성해서 넘길 땐, 해당 코드를 수행할 때마다 익명 객체가 새로 생성 되지만, 람다를 사용할 땐 내부적으로 생성된 1개의 인스턴스를 재사용 한다(람다가 변수를 포획했을 때는 매번 생성 -> 포획할 변수가 매 번 바뀔 수 있으니까)
* 이 동작은 JAVA 메소드를 코틀린에서 호출할 때 쓰이는 방식이고, 대부분의 코틀린 함수들은 람다를 넘겼을 때 다른 방식으로 동작한다(익명 객체를 생성하거나 하진 않는다. 코틀린에는 함수타입이 따로 있기 때문)

**5) SAM 생성자란?**

* 컴파일러가 자동으로 람다를 함수형 인터페이스 무명 클래스로 바꾸지 못하는 경우 사용한다
* JAVA의 함수형 인터페이스의 인스턴를 반환 하고 싶을 경우 사용한다&#x20;
* 컴파일러가 자동 생성 해 준다
* 간편한 코드로 인스턴스를 만들어 사용할 수 있다&#x20;

SAM 생성자를 이용해 인스턴스를 생성한 예제

람다 내용을 인터페이스의 단일 추상 메소드 본문으로 사용한다

단, 'this' 키워드로 인스턴스 자신을 참조할 수 없다

```
    val onClickListInstance = View.OnClickListener { println("onClick") }
```

&#x20;

SAM 생성자를 이용해 인스턴스를 생성한 예제 (인터페이스 직접 생성 해 봄)

자바에서 인터페이스를 정의 하고, 코틀린에서 SAM 생성자를 사용한 것을 볼 수 있다

```
public interface  TestInterface{
    void function_01();
}
```

```
val testInstance = TestInterface { println("Test") }
```

&#x20;

만약 인터페이스에 function\_02()를 추가한다면 아래와 같이 컴파일 에러가 뜬다

에러 내용 : 'Interface TestInterface does not have constructors'

<figure><img src="https://blog.kakaocdn.net/dn/C9b8G/btrm2pqrG70/WVLB4DVkpHztzTMxVeRbgK/img.png" alt="" height="47" width="722"><figcaption></figcaption></figure>

함수형 인터페이스, 즉 SAM 인터페이스에만 SAM 생성자를 지원하는 것을 볼 수 있다

&#x20;

SAM 인터페이스 가 아닐 경우의 인스턴스 생성 예제

* object 키워드를 사용해서 익명 객체 생성으로 사용해야 한다
* Kotlin에서 정의된 인터페이스의 경우엔, SAM 이더라도 object 키워드를 사용해야 한다(Test 해 봄...)
* 'this' 키워드로 인스턴스 자신을 참조할 수 있으므로, 자기 자신이 Listener이고 Listener를 해제해야 하는 경우가 있다면 object 키워드를 사용하면 되겠다

```
val testInterface: TestInterface = object : TestInterface {
    override fun function_01() { println("function01") }
    override fun function_02() { println("function02") }
}
```

&#x20;

SAM 생성자 정리하자면

```
//이렇게 쓸 것을
val obejctInstance = object : View.OnClickListener{
    override fun onClick(v: View?) {
        println("onClick")
    }
}

//SAM 생성자로 간편하게 사용할 수 있다
val lambdaInstance = View.OnClickListener { println("onClick") }
```
