2-5. 플로우 결합

- 여러개의 플로우를 결합해서 사용하는 법을 배워보자

---

- zip으로 묶기
	- 각 플로우마다 값을 하나씩 가져옴
	- 둘 다 하나씩 준비가 될때까지 기다림

```

fun main() = runBlocking<Unit> { 
    val nums = (1..3).asFlow()
    val strs = flowOf("일", "이", "삼") 
    nums.zip(strs) { a, b -> "${a}은(는) $b" }
        .collect { println(it) }
}

결과
1은(는) 일
2은(는) 이
3은(는) 삼

```

---

- combine으로 묶기
	- zip과 달리 한쪽이라도 준비되면 바로 동작을 함
	- 완전히 짝을 맞춰야 하면 zip, 최신의 데이터가 필요하면 combine을 쓰자

```

fun main() = runBlocking<Unit> { 
    val nums = (1..3).asFlow().onEach { delay(100L) }
    val strs = flowOf("일", "이", "삼").onEach { delay(200L) }
    nums.combine(strs) { a, b -> "${a}은(는) $b" }
        .collect { println(it) }
}

결과
1은(는) 일
2은(는) 일
3은(는) 일
3은(는) 이
3은(는) 삼

```


----------------------------------------
