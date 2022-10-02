- 예외나 동작이 끝난 이후의 완료된 동작을 할 필요가 있다

---

- finally 블록
	- try catch 뒤에 나오는 그거 맞음
	- 예외가 발생하던 안하던 무조건 실행하는 구문
	- 명령형 방법이라고 부름

```

fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value -> println(value) }
    } finally {
        println("Done")
    }
}

결과
1
2
3
Done

```

---

- 선언적으로 완료 처리하기
	- 예외처리도 선언적으로 할 수 있는 것처럼 완료도 선언적으로 할 수 있다
	- onCompletion 연산자를 선언해서 완료를 처리할 수 있다
		- 완료에 대한 처리이지 예외를 처리한다고는 안했음. 예외 처리는 catch로 해줘야 함

```

fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking<Unit> {
    simple()
        .onCompletion { println("Done") }
        .collect { value -> println(value) }
}

결과
1
2
3
Done

```

---

- onCompletion의 장점
	- onCompletion은 종료 처리를 할 때 예외가 발생되었는지 여부를 알 수 있다

```

fun simple(): Flow<Int> = flow {
    emit(1)
    throw RuntimeException()
}

fun main() = runBlocking<Unit> {
    simple()
        .onCompletion { cause -> if (cause != null) println("Flow completed exceptionally") } // 예외가 발생한 경우 cause에 널이 아닌 값이 들어감. 발생 안하면 null임
        .catch { cause -> println("Caught exception") }
        .collect { value -> println(value) }
}

결과
1
Flow completed exceptionally
Caught exception

```
