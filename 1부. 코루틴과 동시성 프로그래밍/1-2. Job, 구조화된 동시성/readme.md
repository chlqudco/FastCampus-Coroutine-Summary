- 스코프와 잡의 개념을 알아보자

- 코루틴 빌더를 suspend 안에서 호출하면 어떻게 될까?

```
suspend fun doOneTwoThree() {
    launch {
        println("launch1: ${Thread.currentThread().name}")
        delay(1000L)
        println("3!")
    }

    launch {
        println("launch2: ${Thread.currentThread().name}")
        println("1!")
    }

    launch {
        println("launch3: ${Thread.currentThread().name}")
        delay(500L)
        println("2!")  
    }
    println("4!")
}

fun main() = runBlocking {
    doOneTwoThree()
    println("runBlocking: ${Thread.currentThread().name}")
    println("5!")
}

결과
에러 발생

Suspension functions can be called only within coroutine body
```

- 코루틴 빌더는 코루틴 스코프 안에서만 호출해야 함
	- suspend 함수는 그냥 잠들 수 있다 라는 뜻
	- 해결하기 위해 코루틴 스코프를 만들어 주면 된다

- 가장 간단한 코루틴 스코프를 만드는 방법은 스코프 빌더를 이용하는 것이다
	- 스코프 빌더는 coroutineScope로 만들 수 있다.

```
suspend fun doOneTwoThree() = coroutineScope {
    launch {
        println("launch1: ${Thread.currentThread().name}")
        delay(1000L)
        println("3!")
    }

    launch {
        println("launch2: ${Thread.currentThread().name}")
        println("1!")
    }

    launch {
        println("launch3: ${Thread.currentThread().name}")
        delay(500L)
        println("2!")  
    }
    println("4!")
}

fun main() = runBlocking {
    doOneTwoThree()
    println("runBlocking: ${Thread.currentThread().name}")
    println("5!")
}

결과
4!
launch1: main @coroutine#2
launch2: main @coroutine#3
1!
launch3: main @coroutine#4
2!
3!
runBlocking: main @coroutine#1
5!
```

- 결론 : suspend 함수 안에서 코루틴 빌더를 만들고 싶으면 스코프를 만들어라

- 코루틴 스코프는 runBlocking의 모양과 거의 비슷하다

- withContext 라는 애도 있긴 하다.

- 코루틴 빌더는 반환값을 가지는 경우도 많다.
	- launch는 Job객체를 반환한다
	- 이걸 이용하면 코루틴을 컨트롤 할 수 있다.
	- join은 launch 블럭이 끝날 때 까지 기다리게 하는 것이다.
		- 따라서 suspension point가 된다.
		- delay를 만나도 runBlocking 마냥 양보를 하지 않는다

```
suspend fun doOneTwoThree() = coroutineScope {
    val job = launch {
        println("launch1: ${Thread.currentThread().name}")
        delay(1000L)
        println("3!")
    }
    job.join()

    launch {
        println("launch2: ${Thread.currentThread().name}")
        println("1!")
    }

    launch {
        println("launch3: ${Thread.currentThread().name}")
        delay(500L)
        println("2!")  
    }
    println("4!")
}

fun main() = runBlocking {
    doOneTwoThree()
    println("runBlocking: ${Thread.currentThread().name}")
    println("5!")
}

결과
launch1: main @coroutine#2
3!
4!
launch2: main @coroutine#3
1!
launch3: main @coroutine#4
2!
runBlocking: main @coroutine#1
5!
```

- 코루틴은 협력적으로 동작하기 때문에 여러 코루틴을 만드는게 cost가 높지 않다
	- 쓰레드를 차지하고 있지 않고 양보하기 때문에 괜찮다.
	- 10만개의 코루틴을 만들어도 괜찮다.
	- 쓰레드로 10만개를 만드는건 거의 불가능

```
suspend fun doOneTwoThree() = coroutineScope {
    val job = launch {
        println("launch1: ${Thread.currentThread().name}")
        delay(1000L)
        println("3!")
    }
    job.join()

    launch {
        println("launch2: ${Thread.currentThread().name}")
        println("1!")
    }

    repeat(1000) {
        launch {
            println("launch3: ${Thread.currentThread().name}")
            delay(500L)
            println("2!")  
        }
    }
    println("4!")
}

fun main() = runBlocking {
    doOneTwoThree()
    println("runBlocking: ${Thread.currentThread().name}")
    println("5!")
}

결과
2!가 매우 많이 출력
```

