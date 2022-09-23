- Flow를 응용하는 법을 알아보자!
	- flow 오퍼레이터를 이용해보자!

- 플로우는 map 연산을 통해 데이터를 가공할 수 있다

```

fun flowSomething(): Flow<Int> = flow {
    repeat(10) {
        emit(Random.nextInt(0, 500))
        delay(10L)
    }
}

fun main() = runBlocking {
    flowSomething().map {
        "$it $it"
    }.collect { value ->
        println(value)
    }
}

결과
468 468
10 10
424 424
250 250
209 209
217 217
199 199
416 416
329 329
114 114


```

- filter
	- 조건에 맞는 요소만 남김
	- filter 뒤의 조건을 술어, predicate 라고 부름

```

fun main() = runBlocking<Unit> {
    (1..20).asFlow().filter {
        (it % 2) == 0
    }.collect {
        println(it)
    }
}

결과
2
4
6
8
10
12
14
16
18
20

```

- filterNot
	- 술어가 아닌 조건을 가져옴

```

fun main() = runBlocking<Unit> {
    (1..20).asFlow().filterNot {
        (it % 2) == 0
    }.collect {
        println(it)
    }
}

결과
1
3
5
7
9
11
13
15
17
19

```

- 연속해서 두개 이상의 오퍼레이터도 쓸 수 있다.

```

fun main() = runBlocking<Unit> {
    (1..20).asFlow().filterNot {
        (it % 2) == 0
    }.map{
    	it * 3    
    }.collect {
        println(it)
    }
}

결과
3
9
15
21
27
33
39
45
51
57

```

---

- transform 연산자
	- 조금 더 유연하게 스트림을 변형할 수 있다.

```

suspend fun someCalc(i: Int): Int {
    delay(10L)
    return i * 2
}

fun main() = runBlocking<Unit> {
    (1..20).asFlow().transform {
        emit(it)
        emit(someCalc(it))
    }.collect {
        println(it)
    }
}

결과
1
2
2
4
3
6
4
8
5
10
6
12
7
14
8
16
9
18
10
20
11
22
12
24
13
26
14
28
15
30
16
32
17
34
18
36
19
38
20
40

```

---

- take 연산자
	- 원하는 개수만큼만 결과를 방출함

```

suspend fun someCalc(i: Int): Int {
    delay(10L)
    return i * 2
}

fun main() = runBlocking<Unit> {
    (1..20).asFlow().transform {
        emit(it)
        emit(someCalc(it))
    }.take(5)
    .collect {
        println(it)
    }
}

결과
1
2
2
4
3

```

---

- takeWhile 연산자
	- 개수가 아닌 조건에 따라 가져오는 연산자
	- 원하는 조건이 아닌 경우 바로 종료함

```

suspend fun someCalc(i: Int): Int {
    delay(10L)
    return i * 2
}

fun main() = runBlocking<Unit> {
    (1..20).asFlow().transform {
        emit(it)
        emit(someCalc(it))
    }.takeWhile {
        it < 15
    }.collect {
        println(it)
    }
}

결과
1
2
2
4
3
6
4
8
5
10
6
12
7
14
8

```

---

- drop 연산자
	- 처음 몇개의 결과를 버려버리는 연산자. dropWhile도 있음

```

suspend fun someCalc(i: Int): Int {
    delay(10L)
    return i * 2
}

fun main() = runBlocking<Unit> {
    (1..20).asFlow().transform {
        emit(it)
        emit(someCalc(it))
    }.drop(5)
    .collect {
        println(it)
    }
}

결과
6
4
8
5
10
6
12
7
14
8
16
9
18
10
20
11
22
12
24
13
26
14
28
15
30
16
32
17
34
18
36
19
38
20
40

```
---

- reduce 연산자
	- 흔히 map과 reduce로 함께 소개되는 오래된 매커니즘임
	- 첫번째 값을 결과에 넣은 후에 값을 하나씩 가져와 누진적으로 계산함

```

suspend fun someCalc(i: Int): Int {
    delay(10L)
    return i * 2
}

fun main() = runBlocking<Unit> {
    val value = (1..10)
        .asFlow()
        .reduce { a, b -> // 첫 값은 1과 2, 그 다음엔 3과 3
            a + b
        }
    println(value)
}

결과
55

```

---

- fold 연산자
	- reduce와 매우 흡사하지만 초기값을 가질 수 있다.

```

suspend fun someCalc(i: Int): Int {
    delay(10L)
    return i * 2
}

fun main() = runBlocking<Unit> {
    val value = (1..10)
        .asFlow()
        .fold(10) { a, b ->
            a + b
        }
    println(value)
}

결과
65

```

---

- count 연산자
	- 술어를 만족하는 자료의 개수를 센다.
	- filter는 결과를 걸러낼 뿐인 연산자, count는 결과의 개수를 알려주는 연산자
	- count 같은 놈을 종단 연산자 라고 부름, 특정 값이나 컬렉션등의 결과를 리턴함
	- filter는 중간 연산자라고 부름, 결과를 가져올 수 없는 연산자.
		- collect를 통해 결과를 받아와야 이용할 수 있다
	- 앞에서 배운게 중간인지 종단인지 생각해보자

```

fun main() = runBlocking<Unit> {
    val counter = (1..10)
        .asFlow()
        .count {
            (it % 2) == 0
        }
    println(counter)
}

결과
5

```
