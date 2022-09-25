- 플로우가 어떤 컨텍스트에서 수행되는지 살펴보자

- 플로우는 기본적으로 현재 코루틴 컨텍스트에서 호출된다.
	- 아래 예시가 디폴트 디스패처에서 실행되는걸 볼 수 있음

```

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")
           
fun simple(): Flow<Int> = flow {
    log("flow를 시작합니다.")
    for (i in 1..10) {
        emit(i)
    }
}  

fun main() = runBlocking<Unit> {
    launch(Dispatchers.IO) {
        simple()
            .collect { value -> log("$value를 받음.") } 
    }
}

결과
[DefaultDispatcher-worker-1 @coroutine#2] flow를 시작합니다.
[DefaultDispatcher-worker-1 @coroutine#2] 1를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 2를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 3를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 4를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 5를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 6를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 7를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 8를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 9를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 10를 받음.

```

---

- 플로우를 다른 컨텍스트에서 실행시키면 오류가 생김
	- 플로우 내에서는 컨텍스트를 바꿀 수 없기 때문

```

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")
           
fun simple(): Flow<Int> = flow {
    withContext(Dispatchers.Default) {
        for (i in 1..10) {
            delay(100L)
            emit(i)
        }
    }
}  

fun main() = runBlocking<Unit> {
    launch(Dispatchers.IO) {
        simple()
            .collect { value -> log("${value}를 받음.") } 
    }
}

결과
Exception in thread "main" java.lang.IllegalStateException: Flow invariant is violated:
		Flow was collected in [CoroutineId(2), "coroutine#2":StandaloneCoroutine{Active}@2e2fef93, Dispatchers.IO],
		but emission happened in [CoroutineId(2), "coroutine#2":DispatchedCoroutine{Active}@337f4de3, Dispatchers.Default].
		Please refer to 'flow' documentation or use 'flowOn' instead

```

---

- 결과에 써 있는것처럼 flowOn을 쓰면 컨텍스트를 바꿀 수 있다
	- 업스트림의 컨텍스트만 바꾼다
	- flowOn의 위치가 스트림의 기준이 된다.
	- 아래 예시는 업스트림만 디폴트 디스패쳐로 실행한다
	- flowOn을 여러 군대 붙여도 된다
	- 한번에 여러개를 붙인 경우 맨앞에 붙인게 인정되나?

```

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")
           
fun simple(): Flow<Int> = flow {
    for (i in 1..10) {
        delay(100L)
        log("값 ${i}를 emit합니다.")
        emit(i)
    }  //업스트림
}.flowOn(Dispatchers.Default) //기준

fun main() = runBlocking<Unit> {
    simple().collect { value -> 
        log("${value}를 받음.") // 다운스트림
    } 
}

결과
[DefaultDispatcher-worker-1 @coroutine#2] 값 1를 emit합니다.
[main @coroutine#1] 1를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 값 2를 emit합니다.
[main @coroutine#1] 2를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 값 3를 emit합니다.
[main @coroutine#1] 3를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 값 4를 emit합니다.
[main @coroutine#1] 4를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 값 5를 emit합니다.
[main @coroutine#1] 5를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 값 6를 emit합니다.
[main @coroutine#1] 6를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 값 7를 emit합니다.
[main @coroutine#1] 7를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 값 8를 emit합니다.
[main @coroutine#1] 8를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 값 9를 emit합니다.
[main @coroutine#1] 9를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 값 10를 emit합니다.
[main @coroutine#1] 10를 받음.

```
