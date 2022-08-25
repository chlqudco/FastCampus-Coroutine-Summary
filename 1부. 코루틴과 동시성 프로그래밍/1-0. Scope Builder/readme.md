- 코루틴 빌더를 이용해 코루틴을 만들어 보는 법을 익히자
	- 당연히 시작은 Hello World!

- 코루틴 빌더
	- 코루틴을 만드는 함수
	- 코루틴의 시작은 코루틴 스코프인 것을 잊지 말자
	- runBlocking이란 간단한 코루틴 빌더가 있다
		- 안의 코드가 끝날 때까지 다른 코드를 수행하지 못하게 한다
	- expression body(표현식 형태로 사용할 수 도 있다) 
	- runBlocking 안에서 this를 출력하면 코루틴이 수신 객체 인 것을 알 수 있다.
	- launch 라는 빌더도 있다
		- 당연히 빌더인 만큼 새로운 코루틴을 만듬
		- 할 수 있다면 다른 코드(코루틴 코드 포함)를 같이 수행시키는 빌더이다.
	

```
import kotlinx.coroutines.*

fun main(){
    runBlocking {
       println(Thread.currentThread().name) // 코루틴이 실행되는 쓰레드 이름을 출력, 어느 쓰레드에서 실행중인지 항상 잘 확인하는 습관 필요
       println("Hello")
   }
}

```

```
import kotlinx.coroutines.*

fun main() = runBlocking {
    println(Thread.currentThread().name)
    println("Hello")
}

```

- 코루틴 컨텍스트
	- 코루틴 스코프는 코루틴 컨텍스트를 가지고 있다
	- 코루틴을 처리하기 위한 정보이다.

```
import kotlinx.coroutines.*

fun main() = runBlocking {
    println(coroutineContext)
    println(Thread.currentThread().name)
    println("Hello")
}
```

```
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch {
        println("launch: ${Thread.currentThread().name}")
        println("World!")
    }
    println("runBlocking: ${Thread.currentThread().name}")
    println("Hello")
}
```
- 
- 위 코드 실행시 runBlocking이 모두 실행된 후 launch의 코드가 실행됨
	- 메인 쓰레드를 런브로킹이 먹어버렸기 때문에 기다림

- delay 함수
	- 휴식시간을 주기 위해 사용하는 함수
	- Thread의 sleep과 비슷
	- 코루틴은 해당 쓰레드를 다른 코루틴이 사용하도록 하는 함수

```
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch {
        println("launch: ${Thread.currentThread().name}")
        println("World!")
    }
    println("runBlocking: ${Thread.currentThread().name}")
    delay(500L)
    println("Hello")
}
```

- 딜레이가 있으므로 코루틴은 launch 블록을 실행하러 감

- 딜레이 처럼 쓰레드가 잠드는? 부분을 suspension point 라고 함(매우 중요)

- 코루틴에서 sleep을 쓰면 어떻게 될까

```
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch {
        println("launch: ${Thread.currentThread().name}")
        println("World!")
    }
    println("runBlocking: ${Thread.currentThread().name}")
    Thread.sleep(500)
    println("Hello")
}
```

- 결과 : 런블로킹이 다 실행된 후 launch가 실행 됨
	- 다른 코루틴에게 양보하지 않음(쓰레드를 양보하지 않기 때문)

- 한번에 여러 launch가 있을 때는 어떻게 동작 할까?

```
fun main() = runBlocking {
    launch {
        println("launch1: ${Thread.currentThread().name}")
        delay(1000L)
        println("3!")
    }
    launch {
        println("launch2: ${Thread.currentThread().name}")
        println("1!")
    }
    println("runBlocking: ${Thread.currentThread().name}")
    delay(500L)
    println("2!")
}

결과
runBlocking: main @coroutine#1
launch1: main @coroutine#2
launch2: main @coroutine#3
1!
2!
3!

```

- 앞에 있는 launch 블록 부터 실행이 되는구나
	- 앞서 말한 것 처럼 suspension point가 어디인지 아는게 중요

- 코루틴은 계층적이다
	- 상위 코루틴은 하위 코루틴을 책임짐
	- 아래 예저에서 runBlocking(부모 코루틴)은 모든 launch(자식 코루틴)가 끝날 때 까지 종료되지 않음
	- 따라서 부모를 cancle 하면 자식 코루틴이 모두 cancle 된다

```

fun main() {
    runBlocking {
        launch {
            println("launch1: ${Thread.currentThread().name}")
            delay(1000L)
            println("3!")
        }
        launch {
            println("launch2: ${Thread.currentThread().name}")
            println("1!")
        }
        println("runBlocking: ${Thread.currentThread().name}")
        delay(500L)
        println("2!")
    }
    print("4!")
}

결과
runBlocking: main @coroutine#1
launch1: main @coroutine#2
launch2: main @coroutine#3
1!
2!
3!
4!

```

- suspend 함수
	- 코루틴을 쓰는 함수를 만들기 위해 함수 앞에 suspend를 붙인다
	- 중단 가능한 함수라는 뜻을 가짐
	- 함수로 빼면 가독성이 높아짐

```
import kotlinx.coroutines.*

//delay를 호출하므로 suspend 키워드 사용
suspend fun doThree() {
    println("launch1: ${Thread.currentThread().name}")
    delay(1000L)
    println("3!")
}

//얘는 원칙적으로는 suspend를 안붙어도 됨
suspend fun doOne() {
    println("launch1: ${Thread.currentThread().name}")
    println("1!")
}

suspend fun doTwo() {
    println("runBlocking: ${Thread.currentThread().name}")
    delay(500L)
    println("2!")
}

fun main() = runBlocking {
    launch {
        doThree()
    }
    launch {
        doOne()
    }
    doTwo()
}

결과
runBlocking: main @coroutine#1
launch1: main @coroutine#2
launch1: main @coroutine#3
1!
2!
3!

```

- 런블로킹은 제네릭 타입을 지정할 수 있다
	- 아래 예시는 당연히 main이 Int를 리턴하지 않으므로 잘못된 함수

```
fun main() = runBlocking<Int> {
    launch {
        doThree()
    }
    launch {
        doOne()
    }
    doTwo()
    3
}
```
