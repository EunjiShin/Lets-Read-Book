# 1. 1부 : 좋은 코드

> 안정성, 그리고 가독성

# 안정성

> 💡 오류가 덜 발생하는 코드를 만들기 위해 컴파일러를 활용한다.

## 1. 가변성 제한

- var이나 mutable을 활용해 상태를 가지는 요소를 만들 경우, `일관성 문제, 복잡성 증가 문제가 발생`한다.
    - 상태 추적이 어려워 프로그램의 가독성이 떨어지며, 따라서 오류 발생 확률이 높아진다.
    - 가변성의 특징 때문에 코드 실행 결과를 추론하기 ㅇ려워진다.
    - 멀티스레드 프로그램일 때 동기화가 필요하다.
    - 테스트하기 어렵다.
    - 상태 변경이 발생할 때, 다른 곳에 변경을 전파해야 하는 경우가 존재한다 (리스트 추가에 따른 재정렬 등)
- 따라서 다음 방법을 이용해 가변성을 제어한다.
    - 읽기 전용 프로퍼티 val
        - ≠ immutable. 읽기 전용이지 불변이 아님! 불변을 원하면 final을 쓰자.
        - 프로퍼티에 mutable 객체가 담겨있으면 내부적으로 변할 수 있음
    - 가변 컬렉션과 읽기 전용 컬렉션 구분
        - mutable이 붙은 인터페이스는 읽기 전용 인터페이스를 상속받아 변경 메서드를 추가한 것!
        - 불변이 아닌 읽기 전용임. 겉으로 불변처럼 보이게 한 것 뿐!
            - 읽기 전용이기에 내부 인터페이스에서 실제 컬렉션을 리턴하는 등 유연하게 쓸 수 있다
            - 읽기 전용 컬렉션을 mutable로 다운캐스팅하면 안됨! 필요하다면 copy해서 mutable을 만들것
    - 데이터 클래스의 copy
        - mutable은 예측하기 어렵고 위험하다는 단점이, immutable은 값을 바꿀 수 없다는 단점이 있다.
            - immutable의 장점
                - 상태가 유지되므로 코드 이해하기 쉽고, 병렬 처리할 때 충돌 위험이 없고, 쉽게 캐시할 수 있다.
                - 방어적 복사본을 만들 필요가 없고, 깊은 복사를 할 필요가 없다.
                - mutable이나 immutable 객체를 새로 만들 때 활용하기 쉽다.
                - set이나 map의 key로 사용할 수 있다.
        - 따라서 immutable 객체로 만들되, `자신을 수정한 새로운 객체를 만드는 방식으로 값을 수정`할 수 있는 메서드를 제공하는 방식을 이용한다.
            - java에서 클래스 멤버 변수가 final인 상황을 생각하면 편함!
            - 이때 data 한정자가 제공하는 copy 메서드를 활용하면 편하다. (방어적 복사 활용)
        - 변경 가능한 컬렉션을 만드는 방법
            1. mutable 컬렉션 생성

                ```kotlin
                // 리스트 구현 내에 변경 가능 지점이 존재
                // 멀티스레드 처리 중 동기화가 되고 있는지 어떤지 알 수 없어 위험함
                val list: MutableList<Int> = mutableListOf()
                ```

            2. ⭐️ `mutable 프로퍼티 생성 (var로 읽고 쓸 수 있는 프로퍼티 생성)`
               :: 사용자 정의 세터 또는 델리케이트를 활용해 변경 추적 가능

                ```kotlin
                // 프로퍼티 자체가 변경 가능 지점
                // 따라서 멀티스레드 환경에서의 안정성이 좀 더 좋다
                var list: List<Int> = listOf()
                ```


## 2. 변수 스코프 최소화

- 상태를 정의할 땐 `변수와 프로퍼티의 스코프를 최소화`하자.
    - 프로퍼티보다는 지역 변수 사용하기
        - 읽기 전용 여부와 관계 없이, 변수 정의할 때 초기화되는 것이 좋다
        - 여러 프로퍼티를 한번에 설정할 때는 구조분해 선언을 쓰자
    - 최대한 좁은 스코프를 갖게 변수 사용하기
        - 프로그램 추적을 쉽게 하기 위함
- 변수 스코프가 넓을 때 생기는 위험성들
    - 람다에서의 변수 캡처 발생 (캡처링) : 시퀀스 내부에서 반복문 처리가 존재하는 경우, 필터링이 지연되어 최종적인 변수 값으로만 동작하는 상황이 발생 가능

## 3. 최대한 플랫폼 타입을 사용하지 말 것

> 플랫폼 타입 : 코틀린이 아닌 다른 프로그래밍 언어에서 넘어, nullable 여부를 알 수 없는 타입
>
- 플랫폼 타입 사용을 지양하고, 만약 사용한다면 `해당 함수가 미래에 null을 리턴할 수 있음`을 염두하자.
    - 설계자가 어노테이션으로 명시하거나, 주석을 달지 않는다면 의도하지 않은 null이 발생할 수 있다.
    - 자바를 코틀린과 함께 사용한다면, @Nullable이나 @NotNull을 붙여서 쓰자.

## 4. inferred 타입으로 리턴하지 말 것

> inferred 타입 : 코틀린이 제공하는 타입 추론에 의해 추론된 타입.
>
- 타입 추론의 위험성
    - 추론 타입은 정확히 오른쪽 피연산자에 맞게 설정되며, 슈퍼클래스 또는 인터페이스로 설정되지 않는다.

        ```kotlin
        open class Animal
        class Zebra: Animal()
        
        fun main() {
        	var animal = Zebre()
        	animal = Animal() // 오류: Type mismatch
        }
        ```

    - 직접 타입을 명시하면 해결할 수 있으나, 라이브러리 등 내가 조작할 수 없는 경우가 존재한다.
        - 이 경우 의도치 않은 문제가 발생했을 때 디버깅이 어려워 문제 파악이 힘들다는 문제도 생긴다.
- 리턴 타입은 API를 잘 모르는 사람에게 중요한 정보이니, `외부에서 확인할 수 있게 명시적으로 지정`하자.
    - 동시에, 외부 API를 만들 때는 반드시 타입을 지정하고, 지정한 타입을 특별한 이유 없이 제거하지 말자

## 5. 예외를 활용해 코드에 제한을 걸 것

- 코틀린이 제공하는 코드 동작 제한 방법
    - require 블록 : argument 제한할 수 있다
    - check 블록 : 상태와 관련된 동작 제한
    - assert 블록 : 어떤 것이 true인지 확인 가능 → 테스트 모드에서만 가능
    - return 또는 throw와 함께 활용하는 Elvis 연산자
- `제한을 걸면 좋은 점`
    - 문서를 읽지 않은 개발자도 문제를 확인할 수 있다.
    - 문제가 생겼을 때 함수가 예상하지 못한 동작을 하지 않고 예외를 throw를 하기에 더 안정적이다.
    - 코드가 자체적으로 어느정도 검사가 되기에 단위 테스트를 줄일 수 있다.
    - 스마트 캐스트를 활용할 수 있다.
- 제한 걸기 예시

  > 사용자가 코드를 제대로 사용할 것이라고 믿기보단, 문제 상황을 예측하고 예외를 throw하자.
  >
    - **타입 시스템을 활용해 아규먼트에 제한 걸기**

        ```kotlin
        // require를 활용해 제한을 확인하고, 만족하지 못한다면 IAE를 throw한다.
        fun factorial(n: Int): Long {
        	require(n >= 0)
        	return if (n <= 1) 1 else factorial(n-1) * n
        }
        ```

    - **구체적인 조건을 만족할 때만 함수를 사용할 수 있을 때, 상태에 제한 걸기**

        ```kotlin
        // check 함수를 활용해 상태와 관련된 제한을 확인하고, 만족하지 못한다면 ISE를 throw한다.
        fun speak(text: String) {
        	check(isInitialized)
        	// ... 
        }
        ```

    - **Assert를 이용해 스스로 구현한 내용을 확인하기**

        ```kotlin
        // 구현을 잘못하거나, 해당 코드를 이후 누군가 변경하여 동작이 달라지는 경우를 예방하자.
        // 단, Assert는 테스트 할 때만 활성화되므로, 정말 중요한 상황이면 check를 쓰자.
        // 어플리케이션 실행시에 assert는 예외를 발생하지 못한다는 것도 알아야 한다!
        class StackTest {
        
        	@Test
        	fun 'Stack pops correct number of elements'() {
        		val stack = Stack(20) { it } 
        		val ret = stack.pop(10)
        		assertEquals(10, ret.size)
        	}
        
        }
        ```

        - 단위 테스트 대신 함수에서 assert를 사용할 때의 장점 (단, 이때도 단위테스트 작성은 필수임)
            - Assert 계열은 코드를 자체 점검하여 더 효율적으로 테스트 할 수 있게 한다
            - 특정 상황이 아닌 모든 상황에 대해 테스트 할 수 있다
            - 실행 시점에 정확히 어떻게 되는지 확인할 수 있다
            - 실제 코드가 더 빠른 시점에 실패하게 만들어, 예상치 못한 동작이 발생했을 때 찾기 쉬워진다
    - **nullability와 스마트 캐스팅**

        ```kotlin
        // require와 check 블록에서 true로 검증되면 이후에도 true로 유지된다고 가정
        // 따라서 타입 비교를 할 때 스마트 캐스트가 작동한다.
        fun changeDress(person: Person) {
        	require(person.outfit is Dress) 
        	val dress: Dress = person.outfit
        	// ... 
        }
        
        // null 확인할 때 유용함
        fun sendEmail(person: Person, message: String) {
        	require(person.email != null) 
        	val email: String = person.email
        }
        ```

        ```kotlin
        // requireNotNull, checkNull을 활용해 변수를 언팩할 수 있다.
        class Person(val email: String?)
        fun validateEmail(email: String) { ... }
        
        fun sendEmail(person: Person, text: String) {
        	val email = requireNotNull(person.email)
        	validateEmail(email)
        }
        ```

        ```kotlin
        // nullability를 목적으로 Elvis 연산자를 활용하면, 읽기도 쉽고 유연성도 높아진다.
        val email: String = person.email ? : return
        
        // null일 때 return/throw와 run 함수를 조합해서 활용하면 중지 사유를 로깅할 수 있다.
        fun sendEmail(person: Person, text: String) {
        	val email: String = person.email ?: run {
        		log("Email not sent, no email address")
        		return
        	}
        }
        ```


## 6. 사용자 정의 오류보다는 표준 오류를 사용할 것

- 표준 라이브러리의 오류는 많은 개발자가 알고 있기에, 다른 사람들이 API를 더 쉽게 배우고 이해할 수 있다.

## 7. 결과 부족이 발생할 경우 null과 Failure를 사용할 것

- 함수가 원하는 결과를 만들어 낼 수 없는 상황들
    - 서버로부터 데이터를 읽으려고 했으나, 인터넷 연결에 실패한 경우
    - 조건에 맞는 첫번째 요소를 찾으려 했으나, 조건에 맞는 요소가 없는 경우 등
- 이런 경우를 처리하는 두 가지 방법
    1. `null 또는 ‘실패를 나타내는 sealed 클래스 (=Failure)’ 리턴 :: 예상되는 오류에 사용`
        1. 예상되는 오류를 표현할 때 명시적이고, 효율적이며, 간단하다.
        2. **추가적인 정보 전달이 필요하면 sealed result 클래스를, 그렇지 않다면  null을 사용**하자.
            1. Q. null을 리턴하는게 좋은걸까?
               코틀린은 null-safe 하긴 하지만, null은 어쨌든 아무 것도 없는 것을 나타내는 값인데 그걸 리턴하는게 의미적으로 옳은가?
    2. `예외를 throw :: 예측하기 어려운 예외적 범위의 오류에 사용`
        1. 예외를 정보 전달의 수단이 아닌, 잘못된 특별한 상황을 나타내야 한다.
            1. 개발자는 예외 전파 과정을 추적할 수 없다.
            2. 코틀린의 모든 예외는 unchecked 예외이기에 사용자가 예외를 처리하지 않을 수도 있다.
            3. 예외는 예외 사항을 처리하기 위해 만들어졌으므로 명시적 테스트만큼 빠르지 않다
            4. try-catch 블럭 내에 위치할 경우, 컴파일러가 할 수 있는 최적화가 제한된다.

## 8. 적절하게 null을 처리할 것

> null = lack of value = 값이 부족하다 = 값이 설정되지 않았거나, 제거되었다
>
- 함수가 null을 리턴한다는 것의 의미는 각각이기에, nullable 처리를 위해 의미가 최대한 명확해야 한다.
    - String.toIntOrNull()은 String을 Int로 변환할 수 없을 때 null을 리턴한다
    - Iterable<T>.firstOrNull(() → Boolean)은 조건에 맞는 요소가 없을 때 null을 리턴한다
- 코틀린이 nullable을 처리하는 세 가지 방법
    1. **안전 호출 (?.), 스마트캐스팅, Elvis 연산자 (?:) 등을 활용하여 안전하게 처리한다.**

        ```kotlin
        // 안전 호출
        printer?.print()
        
        // 스마트 캐스팅
        if (printer != null) printer.print()
        
        // Elvis 연산자 사용
        val printerName = printer?.name ?: "Unnamed"
        ```

        - 방어적 프로그래밍 : null일때는 출력하지 않는 등, 모든 가능성을 올바르게 처리하는 것
        - 공격적 프로그래밍 : 모든 상황을 처리 못 할 때 개발자가 수정하게 만드는 것 (require, check..)
    2. **오류를 throw 한다.**

        ```kotlin
        printer?.print() // printer가 null일 것이라고 예상하지 못할 수 있음
        
        // 개발자가 코드를 보고 당연히 그럴 것이라고 생각하게 만드는 부분에서 강제로 오류 발생시키기
        // throw, !!, requireNotNull, checkNotNull 등을 활용
        fun process(user: User) {
        	requireNotNull(user.name)
        	val context = checkNotNull(context)
        	val networkService = getNetworkService(context) ?:
        		throw NoInternetConnection()
        	networkService.getData {
        		data, userData -> show(data!!, userData!!)
        	}
        }
        ```

        - **not-null assertion (!!) 와 관련된 문제**
            - NPE와 동일하게, 예상하지 못한 null이 등장하는 문제가 발생한다.
            - 변수를 null로 설정하고 이후 !!를 사용할 경우 프로퍼티를 계속 언팩해야 하고, 의미있는 null 값을 가질 가능성을 차단하기에 좋지 못한 방식임
                - 따라서 !!은 nullability가 제대로 표현되지 않는 라이브러리를 사용하는 등에서만 사용할 것
            - `lateinit 또는 Delegates.notNull을 활용`하도록 하자.
    3. **함수 또는 프로퍼티를 리팩토링하여 nullable이 나오지 않게 바꾼다.**
        1. nullability는 어떻게든 처리가 필요하므로 비용이 발생한다. 따라서 아예 피하는게 좋음
        - **nullability 회피 방법**
            - nullability에 따라 여러 함수 만들어 사용하기. null, Failure 사용 등!
            - 어떤 값이 클래스 생성 이후에 확실히 설정된다는 보장이 있다면, lateinit 프로퍼티나 notNull 델리케이트를 사용할 것.
                - `lateinit 프로퍼티` : 다른 함수보다 먼저 호출되는 함수에서 값을 초기화하는 등, 프로퍼티가 나중에 설정될 것임을 명시할 때 사용한다.
                - `notNull 델리게이트` : JVM에서 기본 타입과 연결된 타입으로 프로퍼티를 초기화하는 경우, lateinit을 사용할 수 없다. 이럴때 프로퍼티 위임을 이용해 지연 초기화한다.
            - 빈 컬렉션 대신 null 리턴하는 것 금지. 빈 컬렉션과 null은 의미가 다르다!
            - nullable enum과 None enum은 다르다.
                - `nullable enum` : 별도로 처리 필요
                - `None enum` : 정의에 없으므로 필요한 경우에 사용하는 쪽에서 추가해서 쓸 수 있음

## 9. use를 이용해 리소스를 닫을 것

- close를 이용해 명시적으로 닫아야 하는 리소스들은 최종적으로 리소스 참조가 사라질 때 GC가 처리한다.
- 단, 굉장히 느리며 비용도 크기에 그냥 `명시적으로 close 해주는게 좋다`.
    - try-finally 블록은 복잡하니까, 표준 라이브러리의 use 함수를 이용하자.
    - useLines를 이용하면 메모리에 파일 내용을 라인 단위로 유지해 대용량 파일을 적절히 처리할 수 있다.

## 10. 단위 테스트를 만들 것

> 안전한 코드를 만드는 궁극적인 방법은 다양하게 테스트 해보는 것!
>
- 사용자 관점 뿐 아니라, 개발자에게도 유용하도록 (= 안정성을 검증할 수 있는) 단위 테스트를 작성한다.
- 단위 테스트가 확인하는 것들
    - 일반적인 유스케이스 (= 해피 패스)
    - 일반적인 오류 케이스와 잠재적인 문제
    - 엣지 케이스와 잘못된 아규먼트 (ex. 피보나치 수에 음의 정수를 넣는 경우 등)
- 단위 테스트의 장점
    - 테스트가 잘 된 요소는 신뢰할 수 있다.
    - 테스트가 잘 만들어져 있다면 리팩터링이 쉽다. 따라서 점점 코드가 발전한다.
    - 수동 테스트보다 단위 테스트가 훨씬 빠르다. 또한 버그 수정 비용도 줄어든다.
- 단위 테스트의 단점
    - 단위 테스트 작성에 시간이 오래 걸린다. 장기적으로는 디버깅 시간 등을 줄여주니 완전 단점은 아님
    - 테스트를 활용할 수 있게 코드를 조정해야 한다.
    - 좋은 단위 테스트를 만드는게 어렵다. TDD나 소프트웨어 테스팅을 이해해야 한다.
        - 복잡한 부분, 계속해서 수정이 일어나는 부분, 비즈니스 로직, 공용 API, 문제가 자주 발생하는 부분, 수정해야 하는 프로덕션 버그 등을 고려하여 테스트를 짜자.

---

# 가독성

> 💡 코틀린은 간결성이 아니라, 가독성 개선을 목표로 하는 프로그래밍 언어다. <br>
> 가독성을 개선해 깨끗하고 의미 있는 코드를 작성하자.

## 11. 가독성을 목표로 설계할 것

> 가독성 = 코드를 읽고 얼마나 빠르게 이해할 수 있는가?
>
- `인지 부하를 줄이는 코드가 가독성이 좋은 코드`다.
- 숙련된 개발자만을 위한 코드는 좋은 코드가 아니다.
    - 특정 언어의 특성에 익숙하지 않다면, 범용적인 관용구 (if-else 등)을 사용하는게 더 좋다.
    - 관용구가 섞일때마다 복잡도는 높아지고, 디버깅이 어려우며, 동시에 수정하기도 힘들어진다.
    - 만약 읽기 비용이 발생한다면, 지불할 만한 가치가 있는지 고민해봐야 한다.

## 12. 연산자 오버로드를 할 때는 의미에 맞게 사용할 것

- 연산자는 정해진 의미를 갖는다. 따라서 오버로딩 할 경우 `해당 연산자의 사용이 제한`된다.
    - ex. !연산자를 factorial 연산으로 오버로딩 할 경우, not 연산을 쓸 수 없게 된다.
    - 이름이 있는 일반 함수를 쓰는 것이 좋으며, 정말 필요하다면 infix 확장 함수 또는 톱 레벨 함수를 써라
- DSL을 설계하는 상황이라면 무시해도 OK

## 13. Unit? 을 리턴하지 말 것

- Unit? = Unit 또는 null 값을 가질 수 있다 = 예측하기 어렵다
- 예시

    ```kotlin
    // showData가 null을, getData가 null이 아닌 값을 리턴할 때 
    // showData와 showError가 모두 호출된다.
    getData()?.let{ view.showData(it) } ?: view.showError()
    
    // 아래 두 코드를 비교하면 1이 훨씬 가독성이 좋다.
    1. if(!keyIsCorrect(key)) return
    2. verifyKey(key) ?: return
    ```


## 14. 변수 타입이 명확하지 않은 경우 확실하게 지정할 것

- 코틀린이 제공하는 타입 추론 시스템은 일종의 타입 숨기기이므로, 읽는 사람에게 정보를 숨긴다.
- 또한 안전을 위해서라도 상황에 맞게 타입을 지정하는 것이 필요하다.

## 15. 리시버를 명시적으로 참조할 것

> 리시버 = 객체 외부의 람다 코드 블럭을 마치 해당 객체 내에서 사용하는 것처럼 쓸 수 있게 해줌
>
- 리시버를 명시하면 코드가 더 길어지지만, `뭔가를 더 자세하게 설명`할 수 있다.
    - 리시버가 명확하지 않다면, 명시적으로 적어서 의미를 뚜렷하게 나타내자.
    - ex. 스코프 내부에 둘 이상의 리시버가 있는 상황

        ```kotlin
        // 리시버 명시 생략한 경우
        class Node(val name: String) {
        	fun makeChild(childName: String) = 
        		create("$name.$childName")
        			.apply { print("Created ${name}") }
        
        	fun create(name:String): Node? = Node(name)
        }
        
        // 리시버 명시한 경우
        // apply 내부에서 this는 Node? 이므로, 언팩하고 사용함
        class Node(val name: String) {
        	fun makeChild(childName: String) = 
        		create("$name.$childName")
        			.apply { print("Created ${this?.name}") }
        
        	fun create(name:String): Node? = Node(name)
        }
        
        fun main() {
        	val node = Node("parent")
        	node.makeChild("child")
        }
        ```

    - nullable 값을 처리할 때는 also나 let을 사용하는게 더 좋다.

        ```kotlin
        class Node(val name: String) {
        	fun makeChild(childName: String) = 
        		create("$name.$childName")
        			.also { print("Created ${it?.name}") }
        
        	fun create(name:String): Node? = Node(name)
        }
        
        fun main() {
        	val node = Node("parent")
        	node.makeChild("child")
        }
        ```

- DSL은 기본적으로 리시버를 명시하지 않지만, DslMarker 메타 어노테이션을 쓰면 외부 리시버를 사용하는 것을 막아, 외부 리시버를 명시적으로 적게 강제할 수 있다.

## 16. 프로퍼티는 동작이 아니라 상태를 나타낼 것

> 프로퍼티는 상태 집합을, 함수는 행동을 나타낸다고 생각하는게 일반적인 관념.
>
- 코틀린의 프로퍼티는 자바 필드와 다르다!
    - var을 사용해 만든 읽고 쓸 수 있는 프로퍼티 (파생 프로퍼티)는 사용자 정의 세터와 게터를 가질 수 있다.
    - 모든 프로퍼티는 디폴트로 캡슐화되어 있다. 개념적으로 접근자에 더 가깝다.
        - var : 게터와 세터
        - val : 게터
    - 따라서 프로퍼티는 본질적으로 함수이므로 확장 프로퍼티를 이용해 함수처럼 쓸 수 있다.
        - 하지만 이 경우 많은 오해를 불러일으킬 수 있다. 관습적인 개념과 달라지기 때문!
- 프로퍼티는 `상태를 나타내거나 설정하기 위한 목적`으로만 쓰여야 하며, 로직을 포함하지 않는게 좋다.
    - 프로퍼티 대신 함수를 쓰는게 좋은 경우
        - 연산비용이 높거나, 복잡도가 O(1)보다 높은 경우
        - 비즈니스 로직이 포함되는 경우
        - 결정적이지 않은 경우 (같은 동작을 연속으로 했을 때 다른 값이 나오는 경우)
        - 변환의 경우 (관습적인 변환 함수가 이미 존재하는 경우)
        - 게터에서 프로퍼티의 상태 변경이 일어나야 하는 경우
    - 함수 대신 프로퍼티를 사용해야 하는 경우
        - 상태를 추출/설정하는 경우

## 17. 이름 있는 아규먼트를 사용할 것

- 이름 있는 아규먼트를 사용하면 `개발자의 의도를 분명하게 나타낼 수 있고, 입력 순서로부터 안전`해진다.
    - 문제 상황

        ```kotlin
        	// joinToString에 대해 모른다면 "|"를 접두사러 생각하는 등, 의미를 헷갈릴 수 있다.
        val text = (1..10).joinToString("|")
        ```

    - 이름있는 파라미터를 사용해도 되지만, 이 경우 코드에서 잘못 사용될 수도 있다. (위치 실수 등)

        ```kotlin
        val separator = "|"
        val text = (1..10).joinToString("|")
        ```

    - 변수와 이름있는 파라미터를 함께 사용한 예

        ```kotlin
        val separator = "|"
        val text = (1..10).joinToString(separator = separator)
        ```

- 이름 있는 아규먼트를 추천하는 경우
    - 디폴트 아규먼트인 경우
        - 함수 이름은 필수 파라미터와 관련되므로, 디폴트 값을 갖는 옵션 파라미터의 설명이 명확하지 않다.
    - 같은 타입의 파라미터가 많은 경우
        - 이 경우 위치를 잘못 입력하는 실수가 잦아질 수 있다.
    - 함수 타입의 파라미터가 있는 경우 (마지막 위치에 배치된 경우 제외)

        ```kotlin
        // before
        val view = linearLayout {
        	text("Click below")
        	button({ /* 1 */ }, { /* 2 */ }) // 어디가 빌더이고 어디가 클릭 리스너인지 헷갈림
        }
        
        // after : 이름 있는 아규먼트 사용하고 위치를 수정한 경우
        val view = linearLayout {
        	text("Click below")
        	button(onClick = { /* 1 */ }) {
        	 /* 2 */ 
        	} 
        }
        ```


## 18. 코딩 컨벤션을 지킬 것

- 컨벤션을 지켜야 남이 내 코드를 볼 때도 편하고, 내가 남의 코드를 볼 때도 편하다.
- 이왕이면 정적 검사기를 이용해 일관성을 강제하자!