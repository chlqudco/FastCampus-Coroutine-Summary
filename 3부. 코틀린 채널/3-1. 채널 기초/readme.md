- 코루틴간의 데이터 교환을 위한 채널
	- 일종의 파이프
	- 한쪽에더 데이터를 넣고 한쪽에서 데이터를 받음

---

- 채널
	- 보내는 쪽에서는 send로 보내고 받는 쪼게서는 receive로 받을 수 있다
	- send와 receive는 서스펜션 포인트이다
	- 받을 사람이 없다면 잠이들고, 받을 데이터가 없다면 잠이든다

```

fun main() = runBlocking<Unit> {
    val channel = Channel<Int>()
    launch {
        for (x in 1..10) {
            channel.send(x)
        }
    }

    repeat(10) {
            println(channel.receive())
    }
    println("완료")
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
완료

```

---

- trySend, tryReceive
	- 서스펜션 포인트가 생기지 않음.
	- 걍 시도를 해볼 뿐임
	- 특별한 경우에만 쓰면 됨

---

- 같은 코루틴에서 채널을 읽고 쓰는 경우
	- send나 receive가 suspension point이라서 서로에게 의존적이기 때문에 같은 코루틴에서 사용하는 것은 위험
	- 아래 예시는 send가 수신자를 기다리며 잠이 든다. 즉, receive를 수행할 수 없기 때문에  데드락이 걸린다
	- 따라서 무한대기가 발생하여 에러가 생김

```

fun main() = runBlocking<Unit> {
    val channel = Channel<Int>()
    launch {
        for (x in 1..10) {
            channel.send(x)
        }

        repeat(10) {
            println(channel.receive())
        }
        println("완료")
    }
}

결과
Evaluation stopped while it's taking too long️

```

---

- 채널 close
	- 더이상 보낼 자료가 없을 때 close로 닫을 수 있다.
	- 채널은 for in 을 이용해서 반복적으로 receive할 수 있고 close되면 for in은 자동으로 종료
	- 아래 예시에서 close 하지 않는다면 위 예시랑 똑같이 무한루프
	- close 하지 않는 경우는 명시적으로 횟수를 지정해서 가져와야 한다. repeat(10) 이런 식으로.

```

fun main() = runBlocking<Unit> {
    val channel = Channel<Int>()
    launch {
        for (x in 1..10) {
            channel.send(x)
        }
        channel.close()
    }

    for (x in channel) {
        println(x)
    }
    println("완료")
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
완료

```

---

- 채널 프로듀서
	- 생산자 소비자 패턴은 유명한 패턴
	- 코루틴에서는 이 패턴을 도와주는 확장함수가 있다
	- 여태까진 channel도 별도로 만들고 전송이나 받기 위해 launch도 따로 만들었지만 produce가 해결해 줌

- produce
	- 코루틴을 만들고 채널을 제공함
	- produce 확장함수 안에는 ProcucerScope가 만들어 짐
		- CoroutineScope 인터페이스와 SendChannel 인터페이스를 함께 상속
		- 따라서 스코프도 있고 채널도 갖고 있음
	- produce를 사용하면 ProducerScope를 상속받은 ProducerCoroutine 코루틴을 얻게 됨

- consumeEach
	- 채널에서 반복적으로 데이터를 받아감

- 추가 설명
	- runBlocking은 BlockingCoroutine을 쓰는데 이는 AbstractCoroutine를 상속받고 있음
	- 결국 코루틴 빌더는 코루틴을 만드는데 이들이 코루틴 스코프이기도 함
	- AbstractCoroutine은 JobSupport, Job(인터페이스), Continuation(인터페이스), CoroutineScope(인터페이스)을 상속받고 있음
	- Continuation은 다음에 무엇을 할지, Job은 제어를 위한 정보와 제어, CoroutineScope는 코루틴 컨텍스트 제공의 역할

```

fun main() = runBlocking<Unit> {
    val oneToTen = produce { //ProducerScope 가 생성, CoroutineScope와 SendChannel 이 합쳐짐
        for (x in 1..10) {
            channel.send(x)
        }
    }

    oneToTen.consumeEach {
        println(it)
    }
    println("완료")
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
완료

```
