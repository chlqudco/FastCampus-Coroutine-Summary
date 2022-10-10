- 채널은 기본으로 서스펜션 포인트를 갖고 있음
	- send나 receive 때문에 잠이 듬
	- 버퍼링을 이용하면 잠이드는걸 방지할 수 있음.

---

- 버퍼
	- 이전 예제에 버퍼를 사용해보자
	- Channel의 생성자는 버퍼의 사이즈를 지정받음
	- 지정하지 않으면 버퍼를 생성하지 않음
	- 아래 예시는 받는측이 받지 않아도 10개의 데이터를 마구 전송할 수 있음

```

fun main() = runBlocking<Unit> {
    val channel = Channel<Int>(10)
    launch {
        for (x in 1..20) {
            println("${x} 전송중")
            channel.send(x)
        }
        channel.close()
    }

    for (x in channel) {
        println("${x} 수신")
        delay(100L)
    }
    println("완료")
}

결과
1 전송중
2 전송중
3 전송중
4 전송중
5 전송중
6 전송중
7 전송중
8 전송중
9 전송중
10 전송중
11 전송중
12 전송중
1 수신
2 수신
13 전송중
3 수신
14 전송중
4 수신
15 전송중
5 수신
16 전송중
6 수신
17 전송중
7 수신
18 전송중
8 수신
19 전송중
9 수신
20 전송중
10 수신
11 수신
12 수신
13 수신
14 수신
15 수신
16 수신
17 수신
18 수신
19 수신
20 수신
완료

```

---

- 랑데뷰
	- 버퍼 사이즈를 랑데뷰로 지정하는건 버퍼를 안준다는 의미임

- 이외에도 사이즈 대신 사용할 수 있는 다른 설정 값이 있습니다.
	- UNLIMITED - 무제한으로 설정
	- CONFLATED - 오래된 값이 지워짐.
		- 처리하지 못한값은 미련없이 버려버림
	- BUFFERED - 64개의 버퍼. 오버플로우가 날 경우엔 suspend 됨

```

fun main() = runBlocking<Unit> {
    val channel = Channel<Int>(Channel.RENDEZVOUS)
    launch {
        for (x in 1..20) {
            println("${x} 전송중")
            channel.send(x)
        }
        channel.close()
    }

    for (x in channel) {
        println("${x} 수신")
        delay(100L)
    }
    println("완료")
}

결과
1 전송중
2 전송중
1 수신
2 수신
3 전송중
3 수신
4 전송중
4 수신
5 전송중
5 수신
6 전송중
6 수신
7 전송중
7 수신
8 전송중
8 수신
9 전송중
9 수신
10 전송중
10 수신
11 전송중
11 수신
12 전송중
12 수신
13 전송중
13 수신
14 전송중
14 수신
15 전송중
15 수신
16 전송중
16 수신
17 전송중
17 수신
18 전송중
18 수신
19 전송중
19 수신
20 전송중
20 수신
완료

```

---

- 버퍼 오버플로우
	- 버퍼의 오버플로우 정책을 직접 정해줄 수 있음
	- SUSPEND : 잠이 들었다 깨어납니다.
	- DROP_OLDEST : 예전 데이터를 지웁니다.
	- DROP_LATEST : 새 데이터를 지웁니다.


```

fun main() = runBlocking<Unit> {
    val channel = Channel<Int>(2, BufferOverflow.DROP_OLDEST)
    launch {
        for (x in 1..50) {
            channel.send(x)
        }
        channel.close()
    }

    delay(500L)

    for (x in channel) {
        println("${x} 수신")
        delay(100L)
    }
    println("완료")
}

결과
49 수신
50 수신
완료

```
