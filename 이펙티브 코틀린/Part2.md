# 2. 2부 : 코드 설계 

> 대규모 프로젝트에 활용할 수 있는 사례 : 재사용성, 추상화 설계, 객체 생성, 클래스 설계

# 재사용성

> 💡 한번 코드를 잘 만들고 재사용하면 굉장히 편리하다. 이를 위해 장기적으로 코드를 개선해보자.

## 19. knowledge를 반복해서 사용하지 말 것

> = 프로젝트에서 이미 있던 코드를 복붙해서 쓰고 있다면, 뭔가 잘못되고 있는 것 = DRY 원칙 = SSOT

- knowledge (의도적인 정보)의 두 종류
    1. 로직 : 프로그램이 어떻게 동작하는지, 그리고 어떻게 보이는지 → 시간 지나면 변함
    2. `공통 알고리즘` : 원하는 동작을 하기 위한 알고리즘 → 한번 정의되면 크게 변하지 않음
- 변화는 우리가 예상하지 못한 곳에서 일어난다. `변하지 않을거라고 믿는 건 오산!`
    - 변화가 생기면 관련된 모든 곳을 고쳐야 하는데, 이 과정에서 변경 누락이 생기면 에러로 이어진다.
    - 따라서 knowledge 반복은 프로젝트 확장성을 막고, 쉽게 깨지게 (fragile) 만든다.
- 코드를 추출해도 되는지 확인하기 위해 단일 책임 원칙 (SRP)를 활용하자.
    - 얼핏보면 knowledge반복처럼 보여도, 실제로 다른 knowledge를 나타낸다면 추출하면 안된다.
    - 따라서 이를 확인하기 위해, SRP를 체크해 각 클래스를 변경하는 이유가 무엇인지를 참고하자.

## 20. 일반적인 알고리즘을 반복해서 구현하지 말 것

> 여기서의 알고리즘 = 수학 연산, 수집 처리 등 별도의 모듈 또는 라이브러리로 분리할 수 있는 부분을 의미

- 이미 있는 알고리즘을 활용할 때의 장점
    - 코드가 짧아진다
    - 코드 작성 속도가 빨라진다
    - 구현을 따로 읽지 않아도, 함수 이름만 보고도 의미를 파악할 수 있다.
    - 직접 구현할 때 발생할 수 있는 실수를 줄일 수 있다.
    - 제작자들이 한 번만 최적화하면, 사용하는 모든 곳이 혜택을 받는다.
- `표준 라이브러리를 활용`하거나, `확장 함수를 이용해 나만의 범용 유틸리티 함수로 만들어` 사용하자.
    - 알고리즘을 추출할 때는 톱레벨 함수, 프로퍼티 위임, 클래스, 확장 함수 등의 방법이 있다.
    - 다음과 같은 이유로 확장 함수 사용을 추천함!
        - 함수는 상태를 유지하지 않으므로 행위를 나타내기 좋다.
        - 톱레벨 함수와 비교했을 때, 확장 함수는 구체적 타입이 있는 객체에만 사용을 제한할 수 있다.
        - 수정할 객체를 아규먼트가 아닌 확장 리시버로 사용하기에 가독성이 좋다.
        - 객체 사용할 때, 자동 완성 기능 등의 제안을 받을 수 있다.

## 21. 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들 것

> 프로퍼티 위임 (델리게이트) : 다른 객체의 메서드를 활용해 프로퍼티의 접근자를 만드는 것.

- 프로퍼티 위임을 사용하면 `일반적인 행위를 추출하여 재사용`할 수 있다.
    - 따라서 타입이 다르지만 내부적으로 거의 같은 처리를 하며, 프로젝트에서 자주 사용될 때 쓰기 좋다.
- 코틀린 stdlib에서 알아두면 좋은 프로퍼티 델리게이터
    - lazy
    - Delegates.observable
    - Delegates.vetoable
    - Delegates.notNull

## 22. 일반적인 알고리즘을 구현할 때는 제네릭을 사용할 것

> 제네릭 함수 : 타입 아규먼트를 사용하는 함수

- 타입 파라미터는 컴파일러에게 타입 정보를 제공해, 프로그램을 안전하게 만들어준다.
    - 구체적인 타입의 서브타입만 사용할 수 있게 타입을 제한한다.

    ```kotlin
    fun <T : Comparable<t>> Iterable<T>.sorted(): List<T> {
    	/* ... */
    }
    
    fun <T. C : MutableCollection<in T>>
    Iterable<T>.toCollection(destination: C): C {
    	/* ... */
    }
    
    class ListAdapter<T: ItemAdaper>() {}
    ```

    - ex. T를 Iterable<Int>의 서브타입으로 제한하면 T 타입을 제공
        - T 기반의 반복 처리 가능
        - 반복 처리 때 사용되는 객체가 Int라는 것을 알 수 있음
    - ex. T를 Comparable<T>로 제한하면 해당 타입을 비교할 수 있다는 걸 알 수 있음
- 타입 파라미터를 이용해 `type-safe 제네릭 알고리즘과 제네릭 객체를 구현해 구체 자료형의 서브타입을 제한`할 수 있다.


## 23. 타입 파라미터의 섀도잉을 피할 것

> 섀도잉 : 프로퍼티와 파라미터가 같은 이름을 가질 때, 지역 파라미터가 외부 스코프의 프로퍼티를 가리는 상황

- 일반적인 섀도잉은 찾기 쉽지만, 타입 파라미터의 섀도잉은 찾기 어렵다.
    - 클래스 타입 파라미터와 함수 타입 파라미터 사이에서 섀도잉이 발생한 경우

        ```kotlin
        interface Tree
        class Birch: Tree
        class Spruce: Tree
        
        // before : Forest와 addTree의 타입 파라미터가 독립적으로 동작한다.
        class Forest<T: Tree> {
        	fun <T: Tree> addTree(tree: T) {
        		// ...
        	}
        }
        
        // after : 클래스 타입 파라미터 T를 이용해 수정
        class Forest<T: Tree> {
        	fun addTree(tree: T) {
        		// ...
        	}
        }
        
        // after2 : 독립적인 타입 파라미터를 의도했다면, 아예 이름을 다르게 사용해라.
        class Forest<T: Tree> {
        	fun <ST: T> addTree(tree: ST) {
        		// ...
        	}
        }
        ```


## 24. 제네릭 타입과 variance 한정자를 활용할 것

> variance 한정자가 없음 = invariant = 불공변성 = 제네릭 타입으로 만들어지는 타입들이 서로 관련이 없음

- variance 한정자(out, in)을 붙이면 `타입 파라미터가 서로 관련 있게` 제약을 걸어 안전하게 만들 수 있다.
    - **out을 붙여 covariant(공변성)으로 만드는 경우** : A가 B의 서브타입일 때, Cup< A >는 Cup< B >의 서브타입
    - **in을 붙여 contravariant(반공변성)으로 만드는 경우** : A가 B의 서브타입일 때, Cup< A >는 Cup< B >의 슈퍼타입
    - 코틀린 함수의 모든 파라미터 타입은 contravariant이고, 모든 리턴 타입은 covariant이다.
- 코틀린에서의 사용 예시
    - List와 Set의 타입 파라미터는 covariant (out) 이다.
    - Map에서 값의 타입을 나타내는 타입 파라미터는 covariant (out) 이다.
    - Array, MutableList, MutableSet, MutableMap의 타입 파라미터는 invariant 이다.
    - 함수 타입의 파라미터 타입은 contravariant (in) 이다. 리턴 타입은 contravariant (out) 이다.
    - 리턴만 되는 타입은 covariant
    - 허용만 되는 타입은 contravariant

## 25. 공통 모듈을 추출해 여러 플랫폼에서 재사용할 것

> 서로 다른 플랫폼에서 동일한 비즈니스 로직이 있다면, 소스 코드를 재사용할 때 크게 비용을 절감할 수 있다.

- 풀스택 (백-프론트), 모바일 (안드-IOS), 라이브러리 (JVM, JS, native 등) 에서 함께 쓸 수 있도록!
- 공통 모듈을 추출하고, 이후 최대한 공통 모듈을 사용해 여러 플랫폼이 사용할 수 있게 만들자.


---


# 추상화 설계

> 💡 추상화를 통해 복잡성 (구체적인 구현)을 숨기고, 코드를 체계화하고, 사용자의 유연성을 높일 수 있다.

## 26. 함수 내부의 추상화 레벨을 통일할 것

> 컴퓨터 과학에서의 추상화 레벨 : 물리 < 하드웨어 < 어셈블러 < 플밍 언어 < 어플리케이션

> 함수는 작아야 하며, 최소한의 책임만 가져야 한다.

- 추상화 계층이 높을수록 사용이 단순해지지만, 동시에 제어력도 떨어진다.
    - ex. Java는 GC가 자동으로 메모리를 관리해줘 편하지만, 최적화가 어렵다.
    - 추상화 레벨 통일 원칙 (SLA) : 컴퓨터 과학처럼, 함수도 높은 레벨과 낮은 레벨을 구분해야 한다.
        - ex. 한 함수에 많은 로직을 넣는게 아니라, 단계별로 함수를 분리하는게 가독성이 좋다.
- 프로그래밍 중 `별도의 추상화 계층`을 만들되, 각 계층이 너무 커지지 않게 주의하자.

  > 프로그램 아키텍처를 추상화하여 문제 중심으로 프로그래밍 할 수 있다. 이를 통해 서브 시스템의 세부 사항을 숨겨 상호 운영성과 플랫폼 독립성을 얻을 수 있음!
    1. 운영체제 연산과 머신 명령 (제일 하위)
    2. 프로그래밍 언어 구조와 도구
    3. 낮은 레벨 구현 구조
    4. 낮은 레벨 문제 중심
    5. 높은 레벨 문제 중심 (제일 상위)

## 27. 변화로부터 코드를 보호하려면 추상화를 사용할 것

- 추상화를 사용하면 `사용자는 세부 사항을 몰라도 되므로, 추후 코드를 수정해도 영향이 적다`.
- 추상화를 사용한 사례 3가지
    - 상수 : 코드에서 반복되는 리터럴을 상수 프로퍼티로 바꿔 값에 의미를 부여하고 재사용한다.
    - 함수 : 일반적인 알고리즘을 추출하여 코드 반복을 줄이고 유지보수성을 높인다.
    - 클래스 : 상태를 관리하거나, 혹은 상태와 관련된 많은 함수를 관리할 수 있다.
        - 인터페이스를 이용해 사용자에게 클래스(구현)를 숨겨, 더 추상적으로 만들 수 있다.
        - 보편적인 객체를 특수한 객체 (VO)로 래핑하여 자유도를 높일 수 있다.
- 추상화를 사용하면 자유도가 높아지지만, 정의 / 사용 / 이해 / 수정하기는 어려워진다.
    - 너무 많은 것을 숨기면 이해하기 어려워지는 것과 같은 이치
    - 추상화와 가독성 사이의 균형을 맞추기 위해 아래를 고려한다.
        - 팀의 크기, 팀의 경험, 프로젝트의 크기, 특징 세트, 도메인 지식
    - 또는 아래의 규칙을 고려할 수 있다.
        - 많은 개발자가 참여할 경우, 이후 객체 생성과 사용을 변경하기 어려우니 모듈과 부분을 분리하자.
        - 의존성 주입 프레임워크를 사용하면 생성 복잡도는 신경쓰지 않아도 된다.
        - 테스트를 하거나, 타 어플리케이션을 기반으로 만들어야 한다면 추상화를 사용하자.
        - 프로젝트가 작고 실험적이면 추상화를 하지 않아도 괜찮다.

## 28. API 안정성을 확인할 것

- 프로그래밍에서 안정적이고 표준적인 API를 선호하는 이유
    - API가 변경되고 개발자가 업데이트하면 여러 코드를 수동 업데이트해야 한다. 이 과정에서 업데이트로 인한 사이드 이펙트를 피하기 위해 구버전을 유지하는 상황이 발생하고, 이 경우 점점 최신 버전이 제공하는 혜택 또는 보안(버그)를 지원받기 어려워진다.
    - 사용자가 새로운 API를 배워야 한다. 새로운 불안정한 모듈보다는, 안정적인 모듈부터 공부하자.
- 좋은 API를 설계하기 위해 지속적인 업데이트가 필요하다. 이 과정에서 개발자와 사용자의 소통이 중요하다.
    - 시멘틱 버저닝 : 일반적으로 사용하는 버저닝 시스템. 버전 번호를 세 가지로 나누어 관리한다.
        - MAJOR : 호환되지 않는 수준의 API 변경
        - MINOR : 이전 변경과 호환되는 기능 추가
        - PATCH : 간단한 버그 수정

## 29. 외부 API를 랩(wrap)하여 사용할 것

- 불안정한 API를 wrap하여 사용할 때의 장점
    - 문제가 있을 때 wrapper만 변경하면 되므로 API 변경에 쉽게 대응 가능
    - 프로젝트 스타일에 맞춰 API 형태 조정 가능
    - 특정 라이브러리에서 문제가 발생해도 래퍼만 수정하여 타 라이브러리로 교체 가능
    - 필요한 경우 쉽게 동작을 추가하거나 수정 가능
- 단점
    - 래퍼를 따로 정의해야 한다.
    - 다른 개발자가 프로젝트를 다룰 때 어떤 래퍼가 있는지 확인해야 한다.
    - 래퍼들은 프로젝트 내부에서만 유효하기에 문제 발생시 누군가에게 질문하기 어렵다.
- 버전 정보와 사용자 수를 확인하여 API의 안정성을 체크하고, 이후 장단점을 고려해 랩하자.

## 30. 요소의 가시성을 최소화할 것

- 간결한 API를 선호하는 이유
    - 인터페이스가 작을수록 배우고 유지하기 쉽다.
    - 변경할 때는 기존의 것을 숨기는 것보다, 새로운 것을 노출하는게 쉽다.
    - 클래스 상태를 나타내는 프로퍼티를 외부에서 변경할 수 있으면 클래스 상태를 보장할 수 없다.
- 내부적인 변경 없이 작은 인터페이스를 유지하고 싶다면 `가시성을 제한해 변경을 추적`하자.
    - 클래스와 요소를 외부에 노출할 필요가 없다면 가시성 한정자를 통해 외부 접근을 막자.
        - 클래스 멤버의 가시성 한정자 : public(디폴트), private, protected, internal
        - 톱레벨 요소의 가시성 한정자 : public(디폴트), private, internal

## 31. 문서로 규약을 정의할 것

- 일반적으로 함수와 클래스는 이름만으로는 예측할 수 없는 세부 사항을 갖는다. 행위와 요소를 문서화하여 구현에 의존하지 않도록 하자.
- 요소의 규약 : 약속을 기반으로 자유롭게 생각하던 예측을 조정하는 것.
    - 규약을 만들어 사용자와 개발자가 서로에게서 독립되게 하자.
    - 규약은 이름, 주석, 문서, 타입을 이용해 정의한다.
        - 함수 이름과 파라미터만으로 정확하게 표현된다면 주석을 넣을 필요는 없다.
        - 주석을 달 때는 Dokka를 이용해 KDoc 구조로 만들 수 있게 하자.
    - 리스코프 치환 원칙 (LSP)를 지키기 위해 공개 함수에 대한 규약을 잘 지정해야 한다.
        - 문서에 서브클래스에 대한 설명과 규약을 추가하여 사용자가 예측하기 쉽게 하자.
    - 구현의 세부 내용은 달라질 수 잇지만, 캡슐화를 이용해 최대한 보호하는 것이 좋다.

## 32. 추상화 규약을 지킬 것

- 규약은 보증과 같다. 양쪽에서 지켜야 안전하다.
- 만약 규약을 어겨야 하는 경우가 발생할 땐 반드시 문서화하자.

---

# 객체 생성

> 💡 OOP 스타일로 코드를 작성하기 위해 객체 생성 방법을 알아보자.

## 33. 생성자 대신 팩토리 함수를 사용할 것

> 기본 생성자 쓰지 말라 (X). 팩토리 함수는 추가적인 생성자와 대립하는 개념.

- 생성 패턴을 이용해 생성자 대신 `함수로 객체를 생성`하자.
    - 생성자를 이용한 예시

        ```kotlin
        class MyLinkedList<T> (
        	val head: T,
        	val tail: MyLinkedList<T>?
        )
        
        val list = MyLinkedList(1, MyLinkedList(2, null))
        ```

    - 팩토리 함수를 이용한 예시

        ```kotlin
        fun <T> myLinkedListOf(
        	varag elements: T
        ): MyLinkedList<T>? {
        	if(elements.isEmpty()) return null
        	val head = elements.first()
        	val elementsTail = elements.copyOfRange(1, elements.size)
        	val tail = myLinkedListOf(*elementsTail)
        	return MyLinkedList(head, tail)
        }
        
        val list = myLinkedListOf(1, 2)
        ```

- 팩토리 함수의 종류
    - `companion 객체 팩토리 함수 (= 자바의 정적 팩토리 함수) (제일 일반적)`

        ```kotlin
        // 클래스에서 사용한 예시
        class MyLinkedList<T>(
        	val head: T,
        	val tail: MyLinkedList<T>?
        ) {
        	companion object {
        		fun <T> of(varag elements: T): MyLinkedList<T>? {
        			/* ... */
        		}
        	}
        }
        
        // 인터페이스에서 사용한 예시
        class MyLinkedList<T>(
        	val head: T,
        	val tail: MyLinkedList<T>?
        ): MyList<T> {
        	// ... 
        }
        
        interface MyList<T> {
        	// ...
        	
        	companion object {
        		fun <T> of(varag elements: T): MyList<T>? {
        			/* ... */
        		}
        	}
        
        }
        
        // 둘다 사용은 동일!
        val list = MyLinkedList.of(1, 2)
        ```

        - companion 객체로 인터페이스를 구현하거나, 클래스를 상속받을 수도 있다.
        - 추상 companion 객체 팩토리는 값을 가질 수 있으므로, 캐싱을 구현하거나 테스트를 위한 가짜 객체를 만들 수도 있다.
    - 확장 팩토리 함수
        - 이미 companion 객체가 존재할 때, 이 객체의 함수처럼 사용할 수 있는 팩토리 함수가 필요할 때 확장 함수를 사용할 수 있다.

            ```kotlin
            // companion 객체를 직접 수정할 수는 없고, 다른 파일에 함수를 만들어야 한다면 확장함수!
            fun Tool.Companion.createBigTool( /* ... */ ) : BigTool {
            	// ...
            }
            
            interface Tool {
            	companion object { /* ... */ }
            }
            
            // 사용
            Tool.createBigTool()
            ```

        - 외부 라이브러리를 확장할 수 있다.
        - 단, companion 객체가 있어야만 companion 객체를 확장할 수 있음에 유의하자. (비워둬도 ok)
    - 톱레벨 팩토리 함수

        ```kotlin
        class MainActivity: Activity {
        	companion object {
        		fun getIntent(context: Context) = 
        			Intent(context, MainActivity::class.java)
        	}
        }
        
        // Anko 라이브러리를 쓴다면 reified 타입을 활용해 intentFor 함수를 활용할 수 있다.
        intentFor<MainActivity>()
        ```

    - 가짜 생성자

        ```kotlin
        public inline fun<T> List(
        	size: Int,
        	init: (index: Int) -> T
        ): List<T> = MutableList(size, init)
        
        public inline fun <T> MutableList(
        	size: Int,
        	init: (index: Int) -> T
        ): MutableList<T> {
        	val list = ArrayList<T>(size)
        	repeat(size) { index -> list.add(init(index)) }
        	return list
        }
        ```

        - 생성자처럼 보이고 작동하지만, 팩토리 함수와 같은 장점을 얻는 톱레벨 함수
        - 진짜 생성자 대신 사용하는 이유
            - 인터페이스를 위한 생성자가 필요할 때, refied 타입 아규먼트를 갖게하고 싶을 때 사용한다.
            - 단, 위 케이스 제외하면 진짜 생성자가 제공하는 모든 기능을 제공해야 한다.
    - 팩토리 클래스의 메서드

        ```kotlin
        data class Student(
            val id: Int,
            val name: String,
            val surname: String
        )
        
        class StudentsFactory {
            var nextId = 0
            fun next(name: String, surname: String) = Student(nextId++, name, surname)
        }
        
        val factory = StudentsFactory()
        val s1 = factory.next("Marcin", "Moskala")
        ```

        - 상태를 가진다는 점에서 팩토리 함수랑 다름
        - 프로퍼티를 가질 수 있기에 다양하게 최적화 하거나 변용할 수 있다.
- 장점
    - 함수에 이름을 붙일 수 있으므로 이해하기 쉽고, 동일한 파라미터 타입을 갖는 생성자의 충돌 방지 가능
    - 원하는 형태의 타입을 리턴할 수 있으므로, 다른 객체를 생성할 때 사용할 수 있다.
        - 보통 인터페이스 뒤에 실제 객체 구현을 숨길 때 사용!
    - 호출될 때 마다 새로운 객체를 만들 필요가 없다.
        - 싱글톤으로 객체를 하나만 만들거나, 캐싱을 이용해 최적화를 하거나, null을 리턴하게도 할 수 있다.
    - 아직 존재하지 않는 객체 리턴 가능
        - 빌드하지 않고도 객체를 만들거나, 프록시를 통해 만들어지는 객체를 사용할 수 있다.
    - 객체 외부에 팩토리 함수를 만들면 가시성을 제어할 수 있다.
        - 톱레벨 팩토리 함수를 같은 파일 또는 같은 모듈에서만 접근하게 할 수 있다
    - 인라인으로 만들 수 있고, 파라미터를 reified로 만들 수도 있다.
    - 생성자로 만들기 복잡한 객체를 만들 수 있다.
    - 생성자는 슈퍼클래스 또는 기본 생성자를 호출해야 하지만, 팩토리 함수는 원할 때 생성자를 호출할 수 있다.
- 단점
    - 서브 클래스를 생성할 때는 슈퍼클래스의 생성자가 필요하므로, 서브클래스를 만들 수 없다.
    - 이를 보완하기 위해, 슈퍼클래스를 팩토리 함수로 만들었다면 서브클래스도 팩토리 함수로 만든다.

## 34. 기본 생성자에 이름 있는 옵션 아규먼트를 사용할 것

- 생성자와 관련된 자바 패턴인 점층적 생성자 패턴 / 빌더 패턴이 있지만, 디폴트 아규먼트를 갖는 기본 생성자 또는 DSL을 사용하는게 여러모로 좋다.
    - 점층적 생성자 패턴
        - 여러 가지 종류의 생성자를 사용하는 패턴.
        - 코틀린에선 점층적 생성자 대신 **디폴트 아규먼트를 사용하는 것을 추천**한다.

            ```kotlin
            val villagePizze = Pizze(
            	soze = "L",
            	cheese = 1,
            	olives = 2
            )
            ```

            - 파라미터 값을 원하는대로 지정할 수 있고
            - 아규먼트를 원하는 순서대로 지정할 수 있고
            - 명시적으로 아규먼트에 이름을 지정할 수 있기 때문.
    - 빌더 패턴
        - 자바는 이름있는 파라미터와 디폴트 아규먼트를 사용할 수 없기에 빌더 패턴을 쓴다.
        - 팩토리로 사용할 수 있다.
            - 팩토리 메서드를 기본 생성자처럼 쓰려면 커링을 써야 하는데, 코틀린은 커링이 없다.
            - 따라서 데이터 클래스를 이용해, copy로 복제해 사용한다.
        - 코틀린은 그냥 **이름있는 파라미터를 쓰는게 좋다**.
            - 빌더 패턴보다 짧고
            - 더 명확하고
            - 더 쓰기 쉽고
            - 항상 immutable이기에 동시성 문제가 없다.
        - 단, 값의 의미를 묶어서 지정해야 하는 경우, 또는 특정 값을 누적해야 하는 경우엔 빌더 패턴이 좋다.
            - 그래도 많이 쓰이진 않음ㅋㅋ 빌더 패턴을 쓰는 native한 라이브러리를 옮길때나, 디폴트 아규먼트와 DSL을 지원하지 않는 언어에서 사용해야 하는 API를 만들 때 쓰자.

## 35. 복잡한 객체를 생성하기 위한 DSL을 정의할 것

> DSL은 복잡한 객체, 계층 구조를 갖고 있는 객체를 정의할 때 유용하다! 한번 만들고 나면 보일러 플레이트의 복잡성을 숨기면서 개발자의 의도를 나타낼 수 있다.

- 코틀린 DSL은 코틀린의 모든 기능을 활용할 수 있으며, type-safe하다.
- 만약 코틀린 DSL 말고 사용자 정의 DSL을 만들고 싶다면 리시버를 사용하는 함수 타입을 활용하자.


---

# 클래스 설계

## 36. 상속보다는 컴포지션을 활용할 것

> is-a 관계 계층 구조가 명확하지 않을때 상속을 사용하면 곤란함!

> 컴포지션 : 객체를 프로퍼티로 갖고, 함수를 호출하는 형태로 재사용하는 것.

- is-a 관계가 아닌데 코드 추출 또는 재사용이 필요하다면 상속 대신 컴포지션을 사용하자.
    - 단순히 유사한 동작을 한다고 상속 관계로 뽑아내면 이런 문제가 생긴다.
        - 상속은 하나의 클래스만을 대상으로 하므로, 행위를 추출하다보면 깊고 복잡한 계층 구조가 된다.
        - 상속은 클래스의 모든 것을 가져오므로 ISP를 위반한다 (불필요한 함수가 생김!)
        - 슈퍼클래스를 확인해야 하기에 이해하기 어렵다.
        - 내부 구현이 바뀌었을 때 해당 슈퍼 클래스를 상속받는 서브 클래스들이 전부 영향을 받는다.
    - 컴포지션의 장점
        - 필요한 메서드를 관리하는 객체를 다른 객체에 추가하면서, 코드 실행을 명확하게 예측할 수 있다.
        - 객체를 자유롭게 사용할 수 있다.
        - 하나의 클래스 내부에서 여러 기능을 재사용할 수 있다.
        - 우리가 원하는 행위만 가져올 수 있다.
        - 포워딩 메서드(클래스가 인터페이스를 상속받는 동시에, 포함한 객체의 메서드를 이용해 인터페이스 메서드를 구현하는 위임 패턴)를 통해 라이브러리 구현 변경에 의한 사이드 이펙트를 줄이면서 다형성을 유지할 수 있다.
            - 근데 보통 다형성이 그렇게 필요하진 않음. 위임 없는 컴포지션이 훨씬 유연하다는 점 참고!
- 오버라이딩 제한
    - 상속을 허용하지 않는 클래스는 final로 지정하기.
    - 상속은 허용하지만 메서드는 오버라이딩을 허용하지 않는다면 open 키워드 사용하기.
        - open 클래스의 모든 메서드는 오버라이딩 불가
        - open 메서드는 오버라이딩 불가

## 37. 데이터 집합 표현에 data 한정자를 사용할 것

- data 한정자를 이용해 자주 사용되는 함수들을 자동으로 생성할 수 있다.
    - toString, equals와 hashCode, copy, componentN
- tuple 대신 data 클래스를 사용하자
    - 코틀린의 tuple
        - Serializable을 기반으로 만들어지며, toString을 제공하는 제네릭 데이터 클래스
        - 튜플은 데이터 클래스와 같은 역할을 하지만 가독성이 나쁘다.
    - 값에 간단한 이름을 붙일 때, 그리고 미리 알 수 없는 aggregate를 표현할 때를 제외하면 data 클래스를 쓰는 것이 좋다.

    ```kotlin
    // tuple 사용한 예시 -> Pair<String, String>의 의미를 알기 힘들고, 결과도 예측 어려움
    fun String.parseName(): Pair<String, String>? {
    	val indexOfLastSpace = this.trim().lastIndexOf('')
    	if(indexOfLastSpace < 0) return null
    	val firstName = this.take(indexOfLastSpace)
    	val lastName = this.drop(indexOfLastSpace)
    	return Pair(firstName, lastName)
    }
    
    val fullName = "Marcin Moska"
    val (firstName, lastName) = fullName.parseName() ?: return
    
    // data class 사용한 예시
    data class FullName(
    	val firstName: String,
    	val lastName: String
    )
    
    fun String.parseName(): FullName? {
    	val indexOfLastSpace = this.trim().lastIndexOf('')
    	if(indexOfLastSpace < 0) return null
    	val firstName = this.take(indexOfLastSpace)
    	val lastName = this.drop(indexOfLastSpace)
    	return FullName(firstName, lastName)
    }
    
    val fullName = "Marcin Moska"
    val (firstName, lastName) = fullName.parseName() ?: return
    ```

    - 함수 리턴 타입이 명확하고, 더 짧기에 전달하기 쉽고, 데이터 클래스와 다른 이름을 활용해 변수를 해제하면 경고가 출력된다.

## 38. 연산 또는 액션을 전달할 때는 인터페이스 대신 함수 타입을 사용할 것

> SAM(Single-Abstract Method) : 메서드가 하나만 있는 인터페이스. 보통 연산 또는 액션 전달에 활용

- 자바에서 사용하기 위한 API를 설계할 때는 SAM을, 그렇지 않은 경우엔 함수 타입을 사용하자.

    ```kotlin
    // SAM의 예시
    interface OnClick {
    	fun clicked(vies: View)
    }
    
    // SAM을 받은 함수 = 인터페이스를 구현한 객체를 전달받는다.
    fun setOnClickListener(listener: OnClick) {
    	// ...
    }
    
    setOnClickListener(object : OnClick {
    		override fun clicked(view: View) {
    			// ...
    		}
    })
    
    // 위 코드를 함수 타입을 사용하는 코드로 변경한 예시
    fun setOnClickListener(listener: (View) -> Unit) {
    	// ...
    }
    ```

    - 함수 타입을 사용하면 아규먼트에 이름도 붙일 수 있고 (타입 별칭 사용), 아규먼트 분해도 이용할 수 있다.
    - 따라서 인터페이스를 사용해야 하는 명확한 이유가 없다면 함수 타입을 사용하자.


## 39. 태그 클래스보다는 클래스 계층(타입 계층)을 사용하라

> 태그 클래스 = 태그(tag)를 포함한 클래스
태그 = 상수 모드 = 어떤 값이 기준에 만족하는지 확인하는 용도
>

### before: 태그 클래스를 사용한 예

```kotlin
class ValueMatcher<T> private constructor(
    private val value: T? = null,
    private val matcher: Matcher
) {
    fun match(value: T?) = when(matcher) {
        Matcher.EQUAL -> value == this.value
        Matcher.NOT_EQUAL -> value != this.value
        Matcher.LIST_EMPTY -> value is List<*> && value.isEmpty()
        Matcher.LIST_NOT_EMPTY -> value is List<*> && value.isNotEmpty()
    }

    enum class Matcher {
        EQUAL,
        NOT_EQUAL,
        LIST_EMPTY,
        LIST_NOT_EMPTY
    }
    
    companion object {
	    fun <T> equal(value: T) = ValueMatcher<T>(value = value, matcher = Matcher.EQUAL)
	    // 생략
	  }
}
```

- 서로 다른 책임들이 하나의 클래스에 태그로 구분되어 들어가 있다.
    - 한 클래스에 여러 모드를 처리하기 위한 보일러 플레이트가 추가된다.
    - 여러 책임 = 여러 목적. 목적에 따라 프로퍼티가 일관되지 않게 사용될 수 있다.
- 객체 생성을 확인하기 위해 팩터리 메서드를 사용해야 하는 경우가 많다.

### after: sealed 클래스를 이용해 다형성을 활용한 예

> 💡 **왜 하필 sealed 한정자를 사용하는가?** <br>
> : 외부 파일에서 서브클래스를 만드는 행위 자체를 제한하기에, 타입이 추가되지 않는 것이 보장된다. <br>
> : 따라서 when을 만들 때 else 브랜치를 추가로 생성할 필요가 없어 기능 추가가 쉽고, 실수 가능성이 낮다.

```kotlin
sealed class ValueMatcher<T> {
    abstract fun match(value: T): Boolean

    class Equal<T>(val value: T): ValueMatcher<T>() {
        override fun match(value: T): Boolean = value == this.value
    }

    class NotEqual<T>(val value: T): ValueMatcher<T>() {
        override fun match(value: T): Boolean = value != this.value
    }

}
```

- 각각의 모드를 여러개의 클래스로 만들고, 타입 시스템과 다형성을 활용한다.
- 해당 클래스에 sealed 한정자를 붙여 서브클래스 정의를 제한한다.

### **abstract vs sealed**

- abstract
    - 상속과 관련된 설계를 할 때 사용한다
    - 개발자가 새로운 인스턴스를 생성할 수 있다.
- sealed
    - 클래스의 서브 클래스를 제어해야 할 때 사용한다
    - 확장함수를 이용해 클래스에 새로운 함수를 추가할 수 있다.


## 40. equals의 규약을 지켜라

### 동등성의 종류

- 구조적 동등성 (structural equality)
    - equals 메서드와 이를 기반으로 만들어진 == 연산자로 확인하는 동등성
    - a가 nullable이 아닐 때 a == b 는 a.equals(b)와 동일하다.
    - a가 nullable일 때 a?.equals(b) ?: (b === null)과 동일하다.
- 레퍼런스적 동등성 (referential equality)
    - === 연산자로 확인하는 동등성.
    - 두 피연산자가 같은 객체를 가리킬 때 true 리턴

### equals는 왜 필요한가?

- 디폴트 equals와 다른 동작을 해야 하는 경우
    - 모든 클래스는 Any를 상속받고, Any의 equals는 디폴트로 ===을 이용해 인스턴스를 비교한다.
    - 만약 동등성을 다른 형태로 표현해야 한다면, 기본 equals는 유효하지 않다.
- 일부 프로퍼티 만으로 비교해야 하는 경우
- data 한정자를 붙이는 것을 원하지 않거나, 비교 대상 프로퍼티가 기본 생성자에 없는 경우

### equals의 규약

- 반사적 (reflexive) 동작 : x가 null이 아니라면, x.equals(x)는 true를 리턴해야 한다.
- 대칭적 (symmetric) 동작 : x와 y가 null이 아닌 값이라면, x.equals(y)와 y.equals(x)는 동일한 결과를 반환해야 한다.
- 연속적 (transitive) 동작 : x, y, z가 null이 아닌 값이고, x.equals(y)와 y.equals(z)가 true라면, x.equals(z)도 true여야 한다.
- 일관적 (consistent) 동작 : x와 y가 null이 아닌 값이라면, x.equals(y)는 여러 번 실행되더라도 항상 같은 결과를 리턴해야 한다.
    - 단, 비교에 사용되는 프로퍼티를 변경하지 않았을 경우에 한정한다.
- 널과 관련된 동작 : x가 null이 아닌 값이라면, x.equals(null)은 항상 false여야 한다.
- 빠르게 동작해야 한다.

## 41. hashCode의 규약을 지켜라

### hashCode의 규약

- 일관성 유지
    - 어떤 객체를 변경하지 않았다면 (equals 에서 사용된 정보가 바뀌지 않았다면), hashCode는 여러번 호출해도 항상 같은 결과를 반환해야 한다.
- hashCode와 equals는 연관된다
    - equals의 실행 결과로 두 객체가 같다고 나온다면, hashCode 메서드의 호출 결과도 같아야 한다.
    - 즉, 같은 요소는 반드시 같은 해시 코드를 가져야 한다.
    - 따라서 equals를 오버라이드할 때, hashCode도 오버라이드 하는 것이 권장된다.
- hashCode는 최대한 요소를 넓게 퍼뜨려야 한다.
    - 요소가 같은 버킷에 배치되는 것을 최대한 방지하기 위함
    - 같은 버킷에 배치되면 해시 테이블을 쓸 이유가 사라짐!! 중복 제거를 해야 탐색 속도가 빨라질 것

## 42. compareTo의 규약을 지켜랴

> 어떤 객체가  Comparable<T>를 구현하거나, compareTo 메서드를 가지고 있다면, 해당 객체는 어떤 순서를 가지고 있으므로 비교할 수 있다는 의미
>

### compareTo의 동작 규칙

- 비대칭적 동작 : a≥b이고 b≥a라면 a==b이다
- 연속적 동작 : a≥b이고 b≥c 이면 a≥c이다
- 코넥스적 동작 : 두 요소는 어떤 확실한 관계를 가지고 있어야 한다.
    - a ≥ b또는 b ≥ a 중 적어도 하나는 항상 true여야 한다.

### 따로 정의해야 하는가?

- 보통 sortedBy를 이용하면 원하는 키로 정렬할 수 있음
- 여러 프로퍼티 기반 정렬은 sortedWith을 사용하면 됨
- 비교기를 미리 클래스에 companion 객체로 만들어두고 sortedBy 또는 sortedWith에 사용하는 것을 추천

## 43. API의 필수적이지 않는 부분을 확장 함수로 추출하라

### 메서드를 정의할 때, 멤버로 정의할 것인가, 확장 함수로 정의할 것인가?

- 확장 함수는 일반적으로 다른 패키지에 위치하기에, 따로 가져와서 사용해야 한다.
    - 직접 멤버를 추가할 수 없는 경우, 데이터와 행위를 분리하도록 설계된 프로젝트에서 사용됨
    - 따라서 임포트해서 사용해야 하기에, 같은 타입에 같은 이름으로 여러 개 만들 수 있다.
        - 장점 : 여러 라이브러리에서 여러 메서드를 충돌 없이 받을 수 있다
        - 단점 : 같은 이름으로 다른 동작을 하는 확장이 있으므로 위험하다. ⇒ 이런 경우는 멤버 함수를 쓰는게 좋다!
- 확장 함수는 가상(virtual)이 아니다.
    - 즉, 컴파일 시점에 정적으로 선택된다.
    - 따라서 파생 클래스에서 오버라이드 할 수 없다 = 가상 멤버 함수와 다르게 동작한다.
    - 상속을 목적으로 설계되었다면 확장 함수를 쓰면 안된다.
- 멤버가 확장 함수보다 높은 우선 순위를 갖는다.
    - 확장 함수가 첫번째 아규먼트로 리시버가 들어가는 일반 함수로 컴파일되기 때문이다.
- 확장 함수는 클래스가 아닌 타입에 정의하는 것이다.
- 확장은 클래스 레퍼런스에서 멤버로 표시되지 않는다.
    - 따라서 annotation-processor가 따로 처리하지 않는다.

### 요약

- 확장 함수는 더 많은 자유와 유연성을 주지만, 상속, 어노테이션 처리가 지원되지 않으며, 클래스 내부에 없으므로 혼동을 주기 쉽다.

## 44. 멤버 확장 함수의 사용을 피하라

> 어떤 클래스에 대한 확장 함수를 정의할 때, 이를 멤버로 추가하는 것은 좋지 않다.
>

### 이유

- 가시성 제한을 위해 확장 함수를 멤버로 정의하지 마라
    - 단순하게 확장함수를 사용하는 형태를 어렵게 만들 뿐, 가시성을 제한하지는 못한다.
    - 가시성을 제한하고 싶다면 가시성 한정자를 사용하자.
- 레퍼런스를 지원하지 않는다
- 암묵적 접근을 할 때, 어떤 리시버가 선택될지 혼동된다.
- 확장 함수가 외부에 있는 다른 클래스를 리시버로 만들 때, 해당 함수가 어떤 동작을 하는지 명확하지 않다.
- 경험 적은 개발자라면 확장 함수가 직관적이지 않고 어렵다고 느껴질 수 있다.

> 의미가 있다면 사용하되, 단점을 인지하고 사용하자.