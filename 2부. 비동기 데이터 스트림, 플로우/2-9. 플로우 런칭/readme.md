- 플로우를 어떻게 런칭하는지 알아보자

---

- 이벤트를 flow로 처리하기
	- gui 앱은 이벤트가 많이 발생한다
	- 보통 리스너를 달고 콜백에서 처리하곤 한다
	- 리스너 대신 플로우의 onEach를 사용할 수 있다
	- onEach마다 이벤트를 handle 해주면 된다
	- 그러나 collect가 호출 될때까지 기다린다는 단점이 있다.
		- 호출 이전까진 이벤트가 있어도 스트림이 끝날 때까지 기다려야 함
	- 실제로 이벤트는 앱이 끝날 때까지 계속해서 발생함.
		- 따라서 collect는 계속 기다리므로 다음 코드를 진행할 수 없음
		- 리스너는 등록만 해놓고 다음 코드를 처리할 수 있음	
	- 따라서 collect로는 이벤트를 핸들할 수 없음

```

fun events(): Flow<Int> = (1..3).asFlow().onEach { delay(100) }

fun main() = runBlocking<Unit> {
    events()
        .onEach { event -> println("Event: $event") }
        .collect()
    println("Done")
}

결과
Event: 1
Event: 2
Event: 3
Done

```

---

- launchIn을 사용하여 런칭하기
	- collect 대신임
	- launchIn을 이용하면 별도의 코루틴에서 플로우를 런칭할 수 있다
	- 인자로 코루틴 스코프를 전달해야 함, 새로운 코루틴을 만들고 onEach를 돌리는 것
	- 따라서 아래 예시는 Done을 제일 먼저 출력함

```

fun events(): Flow<Int> = (1..3).asFlow().onEach { delay(100) }

fun main() = runBlocking<Unit> {
    events()
        .onEach { event -> println("Event: $event") }
        .launchIn(this)
    println("Done")
}

결과
Done
Event: 1
Event: 2
Event: 3

```

