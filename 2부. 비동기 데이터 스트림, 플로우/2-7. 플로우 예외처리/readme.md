- 플로우도 당연히 예외처리를 할 수 있다.

---

- 수집기 측에서 예외처리 하기
	- 그냥 collect를 try catch로 감싸면 끝

```

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value ->         
            println(value)
            check(value <= 1) { "Collected $value" } // 값이 2가 들어올 때 check에 걸려서 예외가 생김
        }
    } catch (e: Throwable) {
        println("Caught $e")
    } 
}

결과
Emitting 1
1
Emitting 2
2
Caught java.lang.IllegalStateException: Collected 2

```

---

- 다른 형태의 예외처리 방식을 보자.
	- flow 내에 발생한 예외도 try catch로 잡을 수 있다.

```

fun simple(): Flow<String> = 
    flow {
        for (i in 1..3) {
            println("Emitting $i")
            emit(i) // emit next value
        }
    }
    .map { value ->
        check(value <= 1) { "Crashed on $value" }                 
        "string $value"
    }

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value -> println(value) }
    } catch (e: Throwable) {
        println("Caught $e")
    } 
}

결과
Emitting 1
string 1
Emitting 2
Caught java.lang.IllegalStateException: Crashed on 2

```

---

- 예외 투명성
	- 플로우 빌더 코드 블록 내에서 예외를 처리하는건 예외 투명성을 어기는 것이다.
	- 플로우 내부에서 예외를 처리해버리면 밖에서 예외가 발생했는지 알 수가 없기 때문
	- 플로우에서는 catch를 이용해 처리하는 것을 권장한다.
	- catch 블록에서 예외를 새로운 데이터로 만들어 emit을 하거나, 다시 예외를 던지거나, 로그를 남길 수 있다.
	- 아래 예시는 예외 발생시 새로운 데이터를 emit 하고 있다

```

fun simple(): Flow<String> = 
    flow {
        for (i in 1..3) {
            println("Emitting $i")
            emit(i) // emit next value
        }
    }
    .map { value ->
        check(value <= 1) { "Crashed on $value" }                 
        "string $value"
    }

fun main() = runBlocking<Unit> {
    simple()
        .catch { e -> emit("Caught $e") } // emit on exception
        .collect { value -> println(value) }
}

결과
Emitting 1
string 1
Emitting 2
Caught java.lang.IllegalStateException: Crashed on 2

```

---

- catch 투명성
	- catch 연산자는 업스트림(catch 연산자를 쓰기 전의 코드)에만 영향을 미치고 다운스트림에는 영향을 미치지 않는다
	- 이를 catch 투명성이라 한다.
	- 아래 예에서 flow는 catch의 업스트림이다.
	- 하지만 collect는 다운스트림 이므로 예외를 처리하지 못한다

```

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    simple()
        .catch { e -> println("Caught $e") } // does not catch downstream exceptions
        .collect { value -> //collect는 다운스트림
            check(value <= 1) { "Collected $value" }                 
            println(value) 
        }
}

결과
Emitting 1
1
Emitting 2
Exception in thread "main" java.lang.IllegalStateException: Collected 2
 at FileKt$main$1$2.emit (File.kt:15) 
 at FileKt$main$1$2.emit (File.kt:14) 
 at kotlinx.coroutines.flow.FlowKt__ErrorsKt$catchImpl$2.emit (Errors.kt:158) 

```

