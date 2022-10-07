- 파이프 라인을 이용해 두가지 이상의 채널을 연결해서 사용해보자!

---

- 파이프라인
	- 얘도 매우 일반적인 패턴임
	- 하나의 스트림을 프로듀서가 만들고, 다른 코루틴에서 그 스트림을 읽어 새로운 스트림을 만드는 패턴
	- 아래 예시는 채널을 이용해 또다른 채널을 만듬
		- 코루틴스코프의 확장함수로 만듬. 별도의 코루틴을 만들지 않아도 됨
		- 원래는 produce는 코루틴 스코프 안에 있어야 하므로.

```

fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1
    while (true) {
        send(x++)
    }
}

fun CoroutineScope.produceStringNumbers(numbers: ReceiveChannel<Int>): ReceiveChannel<String> = produce {
    for (i in numbers) {
        send("${i}!")
    }
}


fun main() = runBlocking<Unit> {
    val numbers = produceNumbers() //리시브 채널, receive 메서드 호출 가능. send 메서드는 호출 불가능. 원래 채널은 둘다 합친걸 의미
    val stringNumbers = produceStringNumbers(numbers)

    repeat(5) { //close를 어디서도 안했기 때문에 for in 은 사용 불가능
        println(stringNumbers.receive())
    }
    println("완료")
    coroutineContext.cancelChildren()
}

결과
1!
2!
3!
4!
5!
완료


```

---

- 홀수 필터	
	- 파이프라인을 이용한 홀수 필터

```

fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1
    while (true) {
        send(x++)
    }
}

fun CoroutineScope.filterOdd(numbers: ReceiveChannel<Int>): ReceiveChannel<String> = produce { // 여기 자체는 send채널임
    for (i in numbers) {
        if (i % 2 == 1) {
            send("${i}!")
        }
    }
}


fun main() = runBlocking<Unit> {
    val numbers = produceNumbers()
    val oddNumbers = filterOdd(numbers)

    repeat(10) {
        println(oddNumbers.receive())
    }
    println("완료")
    coroutineContext.cancelChildren()
}

결과
1!
3!
5!
7!
9!
11!
13!
15!
17!
19!
완료

```

---

- 소수 필터
	- 누가 이렇게 할까 싶은 예제
	- 이런식으로 쓸 수 있다는 예제

```

fun CoroutineScope.numbersFrom(start: Int) = produce<Int> {
    var x = start
    while (true) {
        send(x++)
    }
}

fun CoroutineScope.filter(numbers: ReceiveChannel<Int>, prime: Int): ReceiveChannel<Int> = produce {
    for (i in numbers) {
        if (i % prime != 0) {
            send(i)
        }
    }
}


fun main() = runBlocking<Unit> {
    var numbers = numbersFrom(2)

    repeat(10) {
        val prime = numbers.receive()
        println(prime)
        numbers = filter(numbers, prime)
    }
    println("완료")
    coroutineContext.cancelChildren()
}

결과
2
3
5
7
11
13
17
19
23
29
완료

```

