- 데이터를 만드는 생산자와 소비자는 같은 속도로 움직일 수 없다.
	- 따라서 데이터를 어딘가에 저장해둘 필요가 있다.
	- 버퍼를 만들어서 유연하게 동작할 수 있다
	- 어떤것이 최선이라고는 말할 수 없다
	- 상황에 맞는 것을 사용하자

---

- 아래 버퍼가 없는 플로우를 먼저 보자
	- 여태 했던 플로우다
	- 생산자 소비자 둘다 매번 기다리니 아주 오래 걸린다

```

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple().collect { value -> 
            delay(300)
            println(value) 
        } 
    }   
    println("Collected in $time ms")
}

결과
1
2
3
Collected in 1228 ms

```

---

- 지연을 최소화 하기 위해 버퍼를 써보자
	- flow(생산자)는 버퍼가 있으면 collect(소비자)가 준비가 되던 안되던 계속 보낼 수 있음 
	- 따라서 전체적인 지연 시간이 줄어듬

```

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple().buffer()
            .collect { value -> 
                delay(300)
                println(value) 
            } 
    }   
    println("Collected in $time ms")
}

결과
1
2
3
Collected in 1091 ms

```

---

- conflate
	- 융합?
	- 중간의 데이터를 누락시켜준다
	- 아래 예시에서 1을 처리하는 도중 2와 3이 들어오지만 2를 버리고 3만 쓰게 된다.

```

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple().conflate()
            .collect { value -> 
                delay(300)
                println(value) 
            } 
    }   
    println("Collected in $time ms")
}

결과
1
3
Collected in 778 ms

```

---

- 마지막값만 처리하는 방법이 있다.
	- 위 예시는 중간값을 누락 했지만 collectLatest는 아예 collect 하는 쪽을 reset 시키는 방법임
	- 마지막 값만 처리하도록 한다.
	- 마지막 값을 기다리는것이 아님
	- 1을 처리하는 중 2가 들어오면 1 하던걸 버리고 2를 처리하기 시작함

```

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple().collectLatest { value -> 
            println("값 ${value}를 처리하기 시작합니다.")
            delay(300)
            println(value) 
            println("처리를 완료하였습니다.")
        } 
    }   
    println("Collected in $time ms")
}

결과
값 1를 처리하기 시작합니다.
값 2를 처리하기 시작합니다.
값 3를 처리하기 시작합니다.
3
처리를 완료하였습니다.
Collected in 707 ms

```
