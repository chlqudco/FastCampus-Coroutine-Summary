- 코루틴 컨텍스트와 디스패처
	- 디스패처를 이용해서 어떤 쓰레드에서 수행될지 지정할 수 있다.
	- 디스패처는 대표적으로 Default, IO, Unconfined, newSingleThreadContext가 있다
	- 대부분 Default나 IO, main이 주로 사용됨

```
fun main() = runBlocking<Unit> {
    launch {
        println("부모의 콘텍스트 / ${Thread.currentThread().name}")
    }

    launch(Dispatchers.Default) {
        println("Default / ${Thread.currentThread().name}")
    }

    launch(Dispatchers.IO) {
        println("IO / ${Thread.currentThread().name}")
    }

    launch(Dispatchers.Unconfined) {
        println("Unconfined / ${Thread.currentThread().name}")
    }

    launch(newSingleThreadContext("Fast Campus")) {
        println("newSingleThreadContext / ${Thread.currentThread().name}")
    }
}

결과
Unconfined / main @coroutine#5
Default / DefaultDispatcher-worker-1 @coroutine#3
IO / DefaultDispatcher-worker-2 @coroutine#4
부모의 콘텍스트 / main @coroutine#2
newSingleThreadContext / Fast Campus @coroutine#6
```

- 순서대로 실행 안된다는게 신기하네

- Unconfined는 메인쓰레드에서 실행 되는 구나
	- 어디에도 속하지 않음.
	- 앞으로 어디에서 수행될지 모르는 장점이자 단점이 있음.

- Default 는 디폴트 어쩌구 worker에서 실행되는 구나
	- 코어 수에 비례하는 스레드 풀에서 수행함
	- ex) 옥타코어 인 경우 8개의 배수인 16개를 갖고 있다고 가정

- IO도 디폴트 어쩌구 worker에서 실행되네?
	- 코어 수보다 훨씬 많은 스레드를 가지는 쓰레드 풀
	- IO 작업은 CPU를 덜 소모하기 때문
	- IO는 2배가 아닌 4~5배를 가질 수 있다
	- 디폴트와 IO는 정책이 다름.
		- 디폴트는 복잡한 연산을 하기 위함
		- 여러개 쓰레드를 만들면 효율적으로 일을 하지 못함
	- 파일 읽기나 네트워크 작업을 할 때 보통 사용

- 아무 디스패처도 안넣으면 부모의 context를 따라가는구나
	- runBlocking이 지금 main에서 실행되므로 main

- newSingleThreadContext 는 지정된 이름이 되는구나
	- 이름처럼 새로운 쓰레드를 만드는구나

---

- async에서 코루틴 디스패쳐 사용하기
	- 디스패처는 대부분의 코루틴 빌더에서 쓸 수 있다.
	- launch에서만 쓸수 있는게 아님

```
fun main() = runBlocking<Unit> {
    async {
        println("부모의 콘텍스트 / ${Thread.currentThread().name}")
    }

    async(Dispatchers.Default) {
        println("Default / ${Thread.currentThread().name}")
    }

    async(Dispatchers.IO) {
        println("IO / ${Thread.currentThread().name}")
    }

    async(Dispatchers.Unconfined) {
        println("Unconfined / ${Thread.currentThread().name}")
    }

    async(newSingleThreadContext("Fast Campus")) {
        println("newSingleThreadContext / ${Thread.currentThread().name}")
    }
}

결과
Default / DefaultDispatcher-worker-1 @coroutine#3
Unconfined / main @coroutine#5
IO / DefaultDispatcher-worker-2 @coroutine#4
부모의 콘텍스트 / main @coroutine#2
newSingleThreadContext / Fast Campus @coroutine#6
```

- 실행 결과가 거의 뭐 당연하게 첫 예제와 동일함
	- UnConfined에 서스펜션 포인트를 넣으면 쓰레드가 바뀜.
	- 테스트 해보자!

```
fun main() = runBlocking<Unit> {
    async(Dispatchers.Unconfined) {
        println("Unconfined / ${Thread.currentThread().name}")
        delay(1000L)
        println("Unconfined / ${Thread.currentThread().name}")
    }
}

결과
Unconfined / main @coroutine#2
Unconfined / kotlinx.coroutines.DefaultExecutor @coroutine#2
```

- 중단점을 만나면 그 다음은 어디서 수행될지 모르는 짜릿함이 있음
	- 쓰지 말라

---

- 부모가 있는 Job과 없는 Job
	- 코루틴은 부모 자식 관계가 성립된다
	- Job()을 만들어 버리면 더이상 부모자식 관계가 아니게 된다
	- Job을 새로 만들어 버리면 누가 부모인지 모르기 때문
	- 따라서 구조적인 동시성이 없어진다

```
fun main() = runBlocking<Unit> { //조부모
    val job = launch { //부모
        launch(Job()) { //자식
            println(coroutineContext[Job])
            println("launch1: ${Thread.currentThread().name}")
            delay(1000L)
            println("3!")
        }

        launch {
            println(coroutineContext[Job])
            println("launch2: ${Thread.currentThread().name}")
            delay(1000L)
            println("1!")
        }
    }

    delay(500L)
    job.cancelAndJoin()
    delay(1000L)
}

결과
"coroutine#3":StandaloneCoroutine{Active}@b684286
launch1: main @coroutine#3
"coroutine#4":StandaloneCoroutine{Active}@36d64342
launch2: main @coroutine#4
3!

```

- 부모를 cancel 했지만 자식이 아니므로 3!이 정상적으로 출력됨

- 그럼 이젠 계층적인 코루틴을 다뤄보자

```

fun main() = runBlocking<Unit> {
    val elapsed = measureTimeMillis {
        val job = launch { // 부모
            launch { // 자식 1
                println("launch1: ${Thread.currentThread().name}")
                delay(5000L)
            }

            launch { // 자식 2
                println("launch2: ${Thread.currentThread().name}")
                delay(10L)
            }
        }
        job.join()
    }
    println(elapsed)
}

결과
launch1: main @coroutine#3
launch2: main @coroutine#4
5077

```

- 부모는 자식 코루틴의 수행이 끝날때까지 기다린다는 걸 명심
	

---

- 코루틴 엘리먼트 결합
	- 코루틴 빌더를 만들 때 인자로 뭔가를 많이 쓸 수 있는데 한번에 쓰는 방법을 배워보자
	- 디스패쳐나 name 등등
	- + 연산으로 합치면 된다
	- 엘리먼트는 배열처럼 조회할 수 있다.

```
@kotlin.ExperimentalStdlibApi
fun main() = runBlocking<Unit> {
    launch {
        launch(Dispatchers.IO + CoroutineName("launch1")) {
            println("launch1: ${Thread.currentThread().name}")
            println(coroutineContext[CoroutineDispatcher])
            println(coroutineContext[CoroutineName])
            delay(5000L)
        }

        launch(Dispatchers.Default + CoroutineName("launch2")) {
            println("launch2: ${Thread.currentThread().name}")
            println(coroutineContext[CoroutineDispatcher])
            println(coroutineContext[CoroutineName])
            delay(10L)
        }
    }
}

결과
launch1: DefaultDispatcher-worker-1 @launch1#3
Dispatchers.IO
CoroutineName(launch1)
launch2: DefaultDispatcher-worker-3 @launch2#4
Dispatchers.Default
CoroutineName(launch2)

```

