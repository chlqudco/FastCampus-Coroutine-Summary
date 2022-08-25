- 취소와 타임아웃
	- 조금 더 복잡한 제어를 알아보자
	- cancle은 코루틴을 취소할 수 있다.

```
suspend fun doOneTwoThree() = coroutineScope {
    val job1 = launch {
        println("launch1: ${Thread.currentThread().name}")
        delay(1000L)
        println("3!")
    }

    val job2 = launch {
        println("launch2: ${Thread.currentThread().name}")
        println("1!")
    }

    val job3 = launch {
        println("launch3: ${Thread.currentThread().name}")
        delay(500L)
        println("2!")  
    }

    delay(800L)
    job1.cancel()
    job2.cancel()
    job3.cancel()
    println("4!")
}

fun main() = runBlocking {
    doOneTwoThree()
    println("runBlocking: ${Thread.currentThread().name}")
    println("5!")
}

결과
launch1: main @coroutine#2
launch2: main @coroutine#3
1!
launch3: main @coroutine#4
2!
4!
runBlocking: main @coroutine#1
5!
```

- 3!은 취소되었기 때문에 실행하지 않음

---

- 취소 불가능한 Job이 있다.

```
suspend fun doCount() = coroutineScope {
    val job1 = launch(Dispatchers.Default) {
        var i = 1
        var nextTime = System.currentTimeMillis() + 100L

        while (i <= 10) {
            val currentTime = System.currentTimeMillis()
            if (currentTime >= nextTime) {
                println(i)
                nextTime = currentTime + 100L
                i++
            }
        }
    }
    
    delay(200L)
    job1.cancel()
    println("doCount Done!")
}

fun main() = runBlocking {
    doCount()
}

결과
1
doCount Done!
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

- launch뒤에 아무것도 없으면 메인쓰레드에서 실행된다
	- 위의 예제처럼 디스패처.디폴트를 설정하면 다른 쓰레드에서 수행됨

- cancel을 했지만 실행되는 이유는 코드가 취소란 개념을 이해하지 못하기 때문
	- Job으로 취소할 수 있는 작업이 아님?
	- 취소가 가능하게 하려면 몇가지 작업을 취소해야 함
	
- 2가지 부분이 문제이다
	- doCount Done이 너무 빨리 출력됨.
	- 취소가 안됨

- 앞의 문제를 먼저 해결해보자
	- cancel뒤에 join을 쓰면 된다
	- 아래 예제는 실제 cancel이 안된다는걸 유의하자

```
suspend fun doCount() = coroutineScope {
    val job1 = launch(Dispatchers.Default) {
        var i = 1
        var nextTime = System.currentTimeMillis() + 100L

        while (i <= 10) {
            val currentTime = System.currentTimeMillis()
            if (currentTime >= nextTime) {
                println(i)
                nextTime = currentTime + 100L
                i++
            }
        }
    }
    
    delay(200L)
    job1.cancel()
    job1.join()
    println("doCount Done!")
}

fun main() = runBlocking {
    doCount()
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
doCount Done!
``

- cancel 뒤에 join을 호출하는 건 자주 일어나기 때문에 한번에 호출할 수 있다.

```
suspend fun doCount() = coroutineScope {
    val job1 = launch(Dispatchers.Default) {
        var i = 1
        var nextTime = System.currentTimeMillis() + 100L

        while (i <= 10) {
            val currentTime = System.currentTimeMillis()
            if (currentTime >= nextTime) {
                println(i)
                nextTime = currentTime + 100L
                i++
            }
        }
    }
    
    delay(200L)
    job1.cancelAndJoin()
    println("doCount Done!")
}

fun main() = runBlocking {
    doCount()
}

결과는 위와 동일
```

- 다음 문제를 해결해보자
	- 코루틴은 isActive를 통해 활성화된 코루틴인지 확인할 수 있다
	- 활성화 된 동안에만 돌게 하면 되겠지요

```
suspend fun doCount() = coroutineScope {
    val job1 = launch(Dispatchers.Default) {
        var i = 1
        var nextTime = System.currentTimeMillis() + 100L

        while (i <= 10 && isActive) {
            val currentTime = System.currentTimeMillis()
            if (currentTime >= nextTime) {
                println(i)
                nextTime = currentTime + 100L
                i++
            }
        }
    }
    
    delay(200L)
    job1.cancelAndJoin()
    println("doCount Done!")
}

fun main() = runBlocking {
    doCount()
}

결과
1
2
doCount Done!
```

- 원하는 대로 잘 동작한다

---

- 취소를 할 때 자원을 해제해야 하는경우가 있을 수 있다
	- ex) file이나 socket을 이용하는 경우
		- 열었으면 반드시 닫아줘야 함
	- launch에서 닫아줘야 하는 경우 finally구문을 이용하면 된다
	- suspend 함수는 취소될 때 JobCancellationException 예외를 발생하기 때문에 try catch를 쓸 수 있다

```
suspend fun doOneTwoThree() = coroutineScope {
    val job1 = launch {
        try {
            println("launch1: ${Thread.currentThread().name}")
            delay(1000L)
            println("3!")
        } finally {
            println("job1 is finishing!")
        }
    }

    val job2 = launch {
        try {
            println("launch2: ${Thread.currentThread().name}")
            delay(1000L)
            println("1!")
        } finally {
            println("job2 is finishing!")
        }
    }

    val job3 = launch {
        try {
            println("launch3: ${Thread.currentThread().name}")
            delay(1000L)
            println("2!")
        } finally {
            println("job3 is finishing!")
        }
    }

    delay(800L)
    job1.cancel()
    job2.cancel()
    job3.cancel()
    println("4!")
}

fun main() = runBlocking {
    doOneTwoThree()
    println("runBlocking: ${Thread.currentThread().name}")
    println("5!")
}

결과
launch1: main @coroutine#2
launch2: main @coroutine#3
launch3: main @coroutine#4
4!
job1 is finishing!
job2 is finishing!
job3 is finishing!
runBlocking: main @coroutine#1
5!

```

- 어떤 코드는 취소가 불가능해야 함
	- withContext(NonCancellable)을 이용해서 취소가 불가능 하게 하자

```
suspend fun doOneTwoThree() = coroutineScope {
    val job1 = launch {
        withContext(NonCancellable) {
            println("launch1: ${Thread.currentThread().name}")
            delay(1000L)
            println("3!")
        }
        delay(1000L)
        print("job1: end")
    }

    val job2 = launch {
        withContext(NonCancellable) {
            println("launch1: ${Thread.currentThread().name}")
            delay(1000L)
            println("1!")
        }
        delay(1000L)
        print("job2: end")
    }

    val job3 = launch {
        withContext(NonCancellable) {
            println("launch1: ${Thread.currentThread().name}")
            delay(1000L)
            println("2!")
        }
        delay(1000L)
        print("job3: end")
    }

    delay(800L)
    job1.cancel()
    job2.cancel()
    job3.cancel()
    println("4!")
}

fun main() = runBlocking {
    doOneTwoThree()
    println("runBlocking: ${Thread.currentThread().name}")
    println("5!")
}

결과
launch1: main @coroutine#2
launch1: main @coroutine#3
launch1: main @coroutine#4
4!
3!
1!
2!
runBlocking: main @coroutine#1
5!

```

- 취소가 안되므로 3!과 1!을 잘 출력 함
	- 자원을 반드시 해제해야 하는 경우에 넣으면 좋음
	- finally는 보장을 하지 못함

---

- 코루틴을 적정한 시간 뒤에 종료하고 싶으면 타임아웃을 이용해야 한다
	- 일정 시간동안 작업이 이루어지지 않으면 취소하는게 맞을 수 있다.

```
suspend fun doCount() = coroutineScope {
    val job1 = launch(Dispatchers.Default) {
        var i = 1
        var nextTime = System.currentTimeMillis() + 100L

        while (i <= 10 && isActive) {
            val currentTime = System.currentTimeMillis()
            if (currentTime >= nextTime) {
                println(i)
                nextTime = currentTime + 100L
                i++
            }
        }
    }
}

fun main() = runBlocking {
    withTimeout(500L) {
        doCount()
    }
}

결과
500L이상 걸린 시점에는 예외가 발생함
1
2
3
4
Exception in thread "main" 

```

- 예외를 잡으려면 try catch를 해야 하는데 귀찮다.
	- 코루틴은 조금 더 간단한 방법을 제공해준다
	- withTimeoutOrNull을 사용하면 타임아웃 될 때 null을 반환한다

```
suspend fun doCount() = coroutineScope {
    val job1 = launch(Dispatchers.Default) {
        var i = 1
        var nextTime = System.currentTimeMillis() + 100L

        while (i <= 10 && isActive) {
            val currentTime = System.currentTimeMillis()
            if (currentTime >= nextTime) {
                println(i)
                nextTime = currentTime + 100L
                i++
            }
        }
    }
}

fun main() = runBlocking {
    val result = withTimeoutOrNull(500L) {
        doCount()
        true
    } ?: false
    println(result)
}

결과
1
2
3
4
false
```
