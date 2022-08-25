- 공유 객체
	- 코루틴은 여러 쓰레드를 사용하기 때문에 동시성 문제가 발생한다
	- 공유 객체를 체계적으로 다뤄보자

- withContext는 코드 블럭을 새로운 코루틴으로 만들어서 동작을 시킴
	- 수행이 완료될 때까지 기다린다.
	- runBlocking은 쓰레드를 붙잡고 있는 것이고 얘는 바깥 코루틴을 멈추게 하는 방식

```
suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100 // 시작할 코루틴의 갯수
    val k = 1000 // 코루틴 내에서 반복할 횟수
    val elapsed = measureTimeMillis {
        coroutineScope { // scope for coroutines
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("$elapsed ms동안 ${n * k}개의 액션을 수행했습니다.")
}

var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            counter++
        }
    }
    println("Counter = $counter")
}

결과
81 ms동안 100000개의 액션을 수행했습니다.
Counter = 94631

```

- 람다 앞에 suspend가 붙어있다.

- 수행 할 때마다 카운터의 값이 달라진다
	- 여러 쓰레드간 서로 다른 정보를 갖고 있기 때문

- 해결해보자!
	- volatile을 적용해보자
	- 가시성을 적용해주는 어노테이션이다.
	- 어떤 쓰레드에서 변경해도 다른 쓰레드에도 영향을 줌

```
suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100 // 시작할 코루틴의 갯수
    val k = 1000 // 코루틴 내에서 반복할 횟수
    val elapsed = measureTimeMillis {
        coroutineScope { // scope for coroutines
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("$elapsed ms동안 ${n * k}개의 액션을 수행했습니다.")
}

@Volatile // 코틀린에서 어노테이션입니다.
var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            counter++
        }
    }
    println("Counter = $counter")
}

결과
55 ms동안 100000개의 액션을 수행했습니다.
Counter = 97772

```

- 10만을 리턴하지 않는 이유는 가시성의 한계때문
	- 값을 변경하는 행위가 동시성 문제를 야기시킴
	- volatile은 가시성만 해결해 줄 뿐 동시에 읽고 수정하는 문제를 해결하지 못함

- 쓰레드 안전한 자료구조를 사용해보자!

```
suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100 // 시작할 코루틴의 갯수
    val k = 1000 // 코루틴 내에서 반복할 횟수
    val elapsed = measureTimeMillis {
        coroutineScope { // scope for coroutines
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("$elapsed ms동안 ${n * k}개의 액션을 수행했습니다.")
}

val counter = AtomicInteger()

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            counter.incrementAndGet()
        }
    }
    println("Counter = $counter")
}

결과
34 ms동안 100000개의 액션을 수행했습니다.
Counter = 100000

````

- AtomicInteger는 특별한 메소드를 제공한다.
	- 한번에 하나의 쓰레드만 접근할 수 있게 해준다.
	- 하지만 모든 문제에서의 정답은 아니다.
		- 사실 모든 상황에서의 정답은 절대 없다.

- 쓰레드 한정을 이용해보자!
	- newSingleThreadContext를 이용하는 방법
	- 하나의 코루틴 컨텍스트를 만듬
	- 항상 같은 쓰레드에서 동작하도록 만듬

```

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100 // 시작할 코루틴의 갯수
    val k = 1000 // 코루틴 내에서 반복할 횟수
    val elapsed = measureTimeMillis {
        coroutineScope { // scope for coroutines
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("$elapsed ms동안 ${n * k}개의 액션을 수행했습니다.")
}

var counter = 0
val counterContext = newSingleThreadContext("CounterContext")

fun main() = runBlocking {
    withContext(counterContext) {
        massiveRun {
            counter++
        }
    }
    println("Counter = $counter")
}

결과
44 ms동안 100000개의 액션을 수행했습니다.
Counter = 100000
```

---

- 뮤텍스
	- 상호배제(Mutual exclusion)의 줄임말
	- 공유 상태를 수정할 때 크리티컬 섹션을 이용하게 한다.
	- 임계영역을 동시에 접근하는 것을 허용하지 않음

```
suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100 // 시작할 코루틴의 갯수
    val k = 1000 // 코루틴 내에서 반복할 횟수
    val elapsed = measureTimeMillis {
        coroutineScope { // scope for coroutines
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("$elapsed ms동안 ${n * k}개의 액션을 수행했습니다.")
}

val mutex = Mutex()
var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            mutex.withLock {
                counter++
            }
        }
    }
    println("Counter = $counter")
}

결과
788 ms동안 100000개의 액션을 수행했습니다.
Counter = 100000

```

- 여태 나온 개념들은 교과서에서나 배울법한 아주 오래된 개념들임
	- 좀 신선한걸 배워보자!

- 액터
	- 1973년에 나온 상대적으로 새로운 개념
	- ? 50년이나 됐는데?
	- 액터가 독점적으로 자료를 가지고, 다른 코루틴과 공유하지 않음
	- 그 자료를 접근하지 위해 액터를 통해서만 접근 가능함
	- 먼저 sealed class를 만들어서 시작함
		- 외부에서 확장이 불가능한 클래스

```
suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100
    val k = 1000
    val elapsed = measureTimeMillis {
        coroutineScope {
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("$elapsed ms동안 ${n * k}개의 액션을 수행했습니다.")  
}

sealed class CounterMsg
object IncCounter : CounterMsg()
class GetCounter(val response: CompletableDeferred<Int>) : CounterMsg()

fun CoroutineScope.counterActor() = actor<CounterMsg> {
    var counter = 0
    for (msg in channel) {
        when (msg) {
            is IncCounter -> counter++
            is GetCounter -> msg.response.complete(counter)
        }
    }
}

fun main() = runBlocking<Unit> {
    val counter = counterActor()
    withContext(Dispatchers.Default) {
        massiveRun {
            counter.send(IncCounter)
        }
    }

    val response = CompletableDeferred<Int>()
    counter.send(GetCounter(response))
    println("Counter = ${response.await()}")
    counter.close()
}

결과
1510 ms동안 100000개의 액션을 수행했습니다.
Counter = 100000

```

- 느린건지 빠른건지 모르겠구만?

