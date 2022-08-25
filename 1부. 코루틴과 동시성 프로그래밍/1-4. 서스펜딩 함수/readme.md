- 서스펜딩 함수
	- 함수 이름 앞에 suspend가 붙은 함수
	- 일단 순차적으로 실행해보자. 일반적인 함수호출과 비슷하다

```
suspend fun getRandom1(): Int {
    delay(1000L)
    return Random.nextInt(0, 500)
}

suspend fun getRandom2(): Int {
    delay(1000L)
    return Random.nextInt(0, 500)
}

fun main() = runBlocking {
    val elapsedTime = measureTimeMillis {
        val value1 = getRandom1()
        val value2 = getRandom2()
        println("${value1} + ${value2} = ${value1 + value2}")
    }
    println(elapsedTime)
}

결과
216 + 313 = 529
2025
```

- 도대체 왜 잘 되는거지? println을 기다린다는게 신기하네. runBlocking이라 그런가

- 총 걸린 시간을 보면 getRandom1이 끝나야 getRandom2가 실행됨
	- 아하 한번에 한 suspend 함수가 수행되는구나!
	- 그럼 동시에 진행시키고 싶으면 어떡하지?
	- 별개의 코루틴을 만들면 되지
	- async 키워드를 이용하면 된다.

- async
	- launch와 비슷한 코루틴 빌더이다
	- 수행 결과를 await 키워드를 통해 받을 수 있다.
	- 따라서 결과를 받아야 하면 async, 받을 필요 없으면 launch를 쓰면 된다

```
suspend fun getRandom1(): Int {
    delay(1000L)
    return Random.nextInt(0, 500)
}

suspend fun getRandom2(): Int {
    delay(1000L)
    return Random.nextInt(0, 500)
}

fun main() = runBlocking {
    val elapsedTime = measureTimeMillis {
        val value1 = async { getRandom1() }
        val value2 = async { getRandom2() }
        println("${value1.await()} + ${value2.await()} = ${value1.await() + value2.await()}")
    }
    println(elapsedTime)
}

결과
341 + 105 = 446
1049
```

- 랜덤 숫자 가져오는 2개의 함수는 동시에 진행된다. printlin에서 value1과 2는 await를 통해 값을 기다린다.
	- await은 join과 결과 가져오기 기능을 동시에 수행한다고 보면 됨
	- 따라서 await()는 suspension point이다
	- async랑 await는 걍 항상 같이 쓴다고 보면 됨

- async를 나중에 수행하고 싶을때는 아래 방법을 사용한다
	- LAZY로 지정한 뒤 start를 호출 해야만 큐에 수행 예약을 한다.

```
suspend fun getRandom1(): Int {
    delay(1000L)
    return Random.nextInt(0, 500)
}

suspend fun getRandom2(): Int {
    delay(1000L)
    return Random.nextInt(0, 500)
}

fun main() = runBlocking {
    val elapsedTime = measureTimeMillis {
        val value1 = async(start = CoroutineStart.LAZY) { getRandom1() }
        val value2 = async(start = CoroutineStart.LAZY) { getRandom2() }

        value1.start()
        value2.start()

        println("${value1.await()} + ${value2.await()} = ${value1.await() + value2.await()}")
    }
    println(elapsedTime)
}

결과
229 + 93 = 322
1032
```

- async를 이용해서 구조적인 동시성을 알아보자
	- 코루틴을 수행 하면서 예외가 발생하면 부모 코루틴과 자식 코루틴 모두에게 예외가 전파된다.
	- 따라서 적절히 처리하지 않으면 취소될 수 있다. 

```
suspend fun getRandom1(): Int {
    try {
        delay(1000L)
        return Random.nextInt(0, 500)
    } finally {
        println("getRandom1 is cancelled.")
    }
}

suspend fun getRandom2(): Int {
    delay(500L)
    throw IllegalStateException()
}

suspend fun doSomething() = coroutineScope { //부모 코루틴
    val value1 = async { getRandom1() } // 자식 코루틴1, 문제 발생의 영향으로 취소
    val value2 = async { getRandom2() } // 자식 코루틴2, 문제 발생
    try {
        println("${value1.await()} + ${value2.await()} = ${value1.await() + value2.await()}")
    } finally {
        println("doSomething is cancelled.")
    }
}

fun main() = runBlocking { // 메인함수 까지 영향을 받게 됨
    try {
        doSomething()
    } catch (e: IllegalStateException) {
        println("doSomething failed: $e")
    }
}

결과
getRandom1 is cancelled.
doSomething is cancelled.
doSomething failed: java.lang.IllegalStateException
```

- getRandom2가 예외를 발생시키므로 부모 코루틴인 coroutineScope 도 영향을 받기 때문에 모든 코루틴이 취소


