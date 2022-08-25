- CEH와 슈퍼 바이저 잡
	- 체계적으로 예외를 관리하는 법을 배워보자

- GlobalScope
	- 어디에도 속하지 않지만 원래부터 존재하는 전역 스코프
	- 지금까지의 코루틴은 다른 코루틴의 자식으로 만들었었다.
	- 이 스코프를 이용하면 코루틴을 쉽게 만들 수 있다
	- 계층적으로 관리가 되지 않는다

```

suspend fun printRandom() {
    delay(500L)
    println(Random.nextInt(0, 500))
}

fun main() {
    val job = GlobalScope.launch(Dispatchers.IO) {
        launch { printRandom() }
    }
    Thread.sleep(1000L)
}

결과
6

```

- Thread.sleep(1000L)를 쓴 까닭은 main이 runBlocking이 아니기 때문
	- delay 함수를 호출할 수 없음

- Global 스코프는 관리하기 힘들다.
	- 일반적으로 잘 안쓴다

---

- CoroutineScope
	- 얘를 써라!
	- 인자로 CoroutineContext를 받는다
		- 엘리먼트를 하나만 넣어도 되고 여러개를 합쳐도 상관 없다

```
suspend fun printRandom() {
    delay(500L)
    println(Random.nextInt(0, 500))
}

fun main() {
    val scope = CoroutineScope(Dispatchers.Default)
    val job = scope.launch(Dispatchers.IO) {
        launch { printRandom() }
    }
    Thread.sleep(1000L)
}

결과
368
```

---

- CEH (코루틴 익셉션 핸들러)
	- 예외를 체계적으로 관리할 수 있다.
	- 핸들러 함수를 만든뒤 등록하자

```
suspend fun printRandom1() {
    delay(1000L)
    println(Random.nextInt(0, 500))
}

suspend fun printRandom2() {
    delay(500L)
    throw ArithmeticException()
}

val ceh = CoroutineExceptionHandler { _, exception ->
    println("Something happend: $exception")
}

fun main() = runBlocking<Unit> {
    val scope = CoroutineScope(Dispatchers.IO)
    val job = scope.launch (ceh) {
        launch { printRandom1() }
        launch { printRandom2() }
    }
    job.join()
}

결과
Something happend: java.lang.ArithmeticException

```

- runBlocking과 CEH
	- 둘은 같이 사용할 수 없다.
	- runBlocking은 자식이 예외로 종료되면 항상 종료되고 CEH를 호출하지 않는다.

```
suspend fun getRandom1(): Int {
    delay(1000L)
    return Random.nextInt(0, 500)
}

suspend fun getRandom2(): Int {
    delay(500L)
    throw ArithmeticException()
}

val ceh = CoroutineExceptionHandler { _, exception ->
    println("Something happend: $exception")
}

fun main() = runBlocking<Unit> {
    val job = launch (ceh) {
        val a = async { getRandom1() }
        val b = async { getRandom2() }
        println(a.await())
        println(b.await())
    }
    job.join()
}

결과
Exception in thread "main" java.lang.ArithmeticException
```

- ceh를 등록해도 호출되지 않음

---

- SupervisorJob
	- SupervisorJob은 예외에 의한 취소를 아래쪽으로 내려가게 해준다
	- 일반적인 Job은 예외가 발생하면 모든 방향의 코루틴을 취소시킨다
	- 그러나 얘는 부모를 취소시키지 않는다

```
suspend fun printRandom1() {
    delay(1000L)
    println(Random.nextInt(0, 500))
}

suspend fun printRandom2() {
    delay(500L)
    throw ArithmeticException()
}

val ceh = CoroutineExceptionHandler { _, exception ->
    println("Something happend: $exception")
}

fun main() = runBlocking<Unit> {
    val scope = CoroutineScope(Dispatchers.IO + SupervisorJob() + ceh)
    val job1 = scope.launch { printRandom1() }
    val job2 = scope.launch { printRandom2() }
    joinAll(job1, job2)
}

결과
Something happend: java.lang.ArithmeticException
270
```

- joinAll을 하면 편리하게 join을 할 수 있다.

- 코루틴스코프와 슈퍼바이저잡을 합친듯 한 SupervisorScope가 있다
	- 사용법은 당연히 코루틴스코프와 비슷함
	- 예외가 발생하는 곳에 반드시 핸들러를 붙여줘야 함
		- 안붙이면 에러 생김

```

suspend fun printRandom1() {
    delay(1000L)
    println(Random.nextInt(0, 500))
}

suspend fun printRandom2() {
    delay(500L)
    throw ArithmeticException()
}

suspend fun supervisoredFunc() = supervisorScope {
    launch { printRandom1() }
    launch(ceh) { printRandom2() }
}

val ceh = CoroutineExceptionHandler { _, exception ->
    println("Something happend: $exception")
}

fun main() = runBlocking<Unit> {
    val scope = CoroutineScope(Dispatchers.IO)
    val job = scope.launch {
        supervisoredFunc()
    }
    job.join()
}

결과
Something happend: java.lang.ArithmeticException
408
```

