- 채널을 1:N 이나 N:1 로 다룰 수 있는 개념을 배워보자

---

- 팬 아웃
	- 여러 코루틴이 동시에 채널을 구독하는 것
	- 한번 가져간 값은 다른녀석이 가져가지 못함

```

fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1
    while (true) {
        send(x++)
        delay(100L)
    }
}

fun CoroutineScope.processNumber(id: Int, channel: ReceiveChannel<Int>) = launch {
    channel.consumeEach {
        println("${id}가 ${it}을 받았습니다.")
    }
}


fun main() = runBlocking<Unit> {
    val producer = produceNumbers()
    repeat (5) {
        processNumber(it, producer)
    }
    delay(1000L)
    producer.cancel()
}

결과
0가 1을 받았습니다.
0가 2을 받았습니다.
1가 3을 받았습니다.
2가 4을 받았습니다.
3가 5을 받았습니다.
4가 6을 받았습니다.
0가 7을 받았습니다.
1가 8을 받았습니다.
2가 9을 받았습니다.
3가 10을 받았습니다.

```

---

- 팬 인
	- 팬아웃과 반대로 생산자가 많은 경우임

```

suspend fun produceNumbers(channel: SendChannel<Int>, from: Int, interval: Long) {
    var x = from
    while (true) {
        channel.send(x)
        x += 2
        delay(interval)
    }
}

fun CoroutineScope.processNumber(channel: ReceiveChannel<Int>) = launch {
    channel.consumeEach {
        println("${it}을 받았습니다.")
    }
}


fun main() = runBlocking<Unit> {
    val channel = Channel<Int>()
    launch {
        produceNumbers(channel, 1, 100L)
    }
    launch {
        produceNumbers(channel, 2, 150L)
    }
    processNumber(channel)
    delay(1000L)
    coroutineContext.cancelChildren()
}

결과
1을 받았습니다.
2을 받았습니다.
3을 받았습니다.
4을 받았습니다.
5을 받았습니다.
6을 받았습니다.
7을 받았습니다.
9을 받았습니다.
8을 받았습니다.
11을 받았습니다.
10을 받았습니다.
13을 받았습니다.
15을 받았습니다.
12을 받았습니다.
17을 받았습니다.
14을 받았습니다.
19을 받았습니다.

```

---

- 공정한 채널
	- 두 개의 코루틴에서 채널을 서로 사용할 때 공정하게 기회를 주는 방법
	- 위의 예시는 순서가 뒤죽박죽 이였음
		- 사실 딜레이 때문임 ㅎㅎ, 딜레이 없으면 공정해짐


```

suspend fun someone(channel: Channel<String>, name: String) {
    for (comment in channel) {
        println("${name}: ${comment}")
        channel.send(comment.drop(1) + comment.first())
        delay(100L)
    }
}

fun main() = runBlocking<Unit> {
    val channel = Channel<String>()
    launch {
        someone(channel, "민준")
    }
    launch {
        someone(channel, "서연")
    }
    channel.send("패스트 캠퍼스")
    delay(1000L)
    coroutineContext.cancelChildren()
}

결과
}
민준: 패스트 캠퍼스
서연: 스트 캠퍼스패
민준: 트 캠퍼스패스
서연:  캠퍼스패스트
민준: 캠퍼스패스트 
서연: 퍼스패스트 캠
민준: 스패스트 캠퍼
서연: 패스트 캠퍼스
민준: 스트 캠퍼스패
서연: 트 캠퍼스패스
민준:  캠퍼스패스트

```

---

- select
	- 요청을 빨리 끝내기 위해 먼저 끝나는 요청을 먼저 처리함
	- 아래 예시는 fast가 빨라서 3번이나 독식한걸 볼 수 있음

```

fun CoroutineScope.sayFast() = produce<String> {
    while (true) {
        delay(100L)
        send("패스트")
    }
}

fun CoroutineScope.sayCampus() = produce<String> {
    while (true) {
        delay(150L)
        send("캠퍼스")
    }
}

fun main() = runBlocking<Unit> {
    val fasts = sayFast()
    val campuses = sayCampus()
    repeat (5) {
        select<Unit> { // 먼저 끝난 애꺼를 듣겠다
            fasts.onReceive {
                println("fast: $it")
            }
            campuses.onReceive {
                println("campus: $it")
            }
        }
    }
    coroutineContext.cancelChildren()
}

결과
fast: 패스트
campus: 캠퍼스
fast: 패스트
fast: 패스트
campus: 캠퍼스

```

- 채널에 대해 onReceive를 사용하는 것 이외에도 아래의 상황에서 사용할 수 있습니다.
	- Job - onJoin
	- Deferred - onAwait
	- SendChannel - onSend
	- ReceiveChannel - onReceive, onReceiveCatching
	- delay - onTimeout
	- 무슨소리지?

