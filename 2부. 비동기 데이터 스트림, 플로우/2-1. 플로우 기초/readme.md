- 여태까지는 순차적으로 프로그래밍할 수 있는 코루틴을 사용
	- 지금부터는 변화추적이 쉬운 스트림을 이용한 플로우를 사용

- Flow는 코루틴으로 만들어진 코틀린에 쓸 수 있는 비동기 스트림
	- 효율적으로 동작함, 원하는 형태로 다른 디스패처를 쓸 수 있음
	- 여태는 순차적, 지금부턴 Flow를 이용해 변화를 추적

- Flow의 타입은 Flow이며 제네릭으로 어떤 타입을 반환할지 선언
	- Flow를 만드는건 간단하게 flow 빌더를 사용하면 됨
	- emit으로 스트림에 값을 방출할 수 있다. 하나의 강에 값을 흘려보내는 걸로 비유가능
	- 스트림은 collect 함수를 통해 값을 받아올 수 있음	
	
- Flow는 콜드 스트림 이다.
	- 요청이 있을 때만 값이 전달이 된다 (collect를 해야만 값이 전달됨)

```

fun flowSomething(): Flow<Int> = flow {
    repeat(10) {
        emit(Random.nextInt(0, 500))
        delay(10L)
    }
}

fun main() = runBlocking {
    flowSomething().collect { value ->
        println(value)
    }
}

결과
442
226
157
435
146
402
237
428
367
391

```

- 플로우 취소
	- withTimeoutOrNull을 이용해 코루틴을 종료시킨 것 처럼 똑같이 사용할 수 있다.

```

fun flowSomething(): Flow<Int> = flow {
    repeat(10) {
        emit(Random.nextInt(0, 500))
        delay(100L)
    }
}

fun main() = runBlocking<Unit> {
    val result = withTimeoutOrNull(500L) {
        flowSomething().collect { value ->
            println(value)
        }
        true
    } ?: false
    if (!result) {
        println("취소되었습니다.")
    }
}

결과
51
165
292
385
65
취소되었습니다.

```

- result로 값을 받지 않는다면 예외가 발생하게 됨

```

fun flowSomething(): Flow<Int> = flow {
    repeat(10) {
        emit(Random.nextInt(0, 500))
        delay(100L)
    }
}

fun main() = runBlocking<Unit> {
    withTimeoutOrNull(500L) {
        flowSomething().collect { value ->
            println(value)
        }
    }
}

결과
329
159
464
135
120

예외가 안생김;;

```

- 다른 플로우 빌더
	- flow 이외에도 flowOf나 asFlow등의 빌더가 있다.
	- flowOf는 여러 값을 인자로 전달해서 플로우를 만들 수 있다.

```

fun main() = runBlocking<Unit> {
    flowOf(1, 2, 3, 4, 5).collect { value ->
        println(value)
    }
}

결과
1
2
3
4
5

```

- 위 코드는 아래 코드와 동일

```

flow{
	emit(1)
	emit(2)
	emit(3)
	emit(4)
	emit(5)
}.collect { println(it) }

```

- asFlow 빌더
	- 컬렉션이나 시퀀스를 전달해 플로우를 만들 수 있다

```

fun main() = runBlocking<Unit> {
    listOf(1, 2, 3, 4, 5).asFlow().collect { value ->
        println(value)
    }
    (6..10).asFlow().collect {
        println(it)
    }
}

결과
1
2
3
4
5
6
7
8
9
10

```
