- 다양한 플로우의 기능을 알아보자

- flatMapConcat
	- 플로우는 3가지 유형의 flatMap을 지원하고 있다
	- flatMapConcat, flatMapMerge, flatMapLatest
	- concat은 첫번째 요소에 대해 플래트닝 하고 두번째 요소를 한다
	- 예시로 보자
	- 1에 대한 처리가 끝날 때 까지 2에대한 처리를 시작하지 않는다

```

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First") 
    delay(500) // wait 500 ms
    emit("$i: Second")    
}

fun main() = runBlocking<Unit> { 
    val startTime = System.currentTimeMillis() // remember the start time 
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
        .flatMapConcat {
            requestFlow(it)
        }                                                                           
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
}

결과
1: First at 127 ms from start
1: Second at 628 ms from start
2: First at 728 ms from start
2: Second at 1229 ms from start
3: First at 1329 ms from start
3: Second at 1830 ms from start

``` 

---

- flatMapMerge는 병렬?적으로 진행된다

```

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First") 
    delay(500) // wait 500 ms
    emit("$i: Second")    
}

fun main() = runBlocking<Unit> { 
    val startTime = System.currentTimeMillis() // remember the start time 
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
        .flatMapMerge {
             requestFlow(it) 
        }                                                                           
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
}

결과
1: First at 172 ms from start
2: First at 268 ms from start
3: First at 369 ms from start
1: Second at 672 ms from start
2: Second at 768 ms from start
3: Second at 870 ms from start

```

---

- flatMapLatest
	- 다음 요소의 플레트닝을 시작하면서 이전에 진행중인 플레트닝은 취소시켜 버림
	- 마지막 요소만 플래트닝 한다고 보면 됨

```

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First") 
    delay(500) // wait 500 ms
    emit("$i: Second")    
}

fun main() = runBlocking<Unit> { 
    val startTime = System.currentTimeMillis() // remember the start time 
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
        .flatMapLatest {
            requestFlow(it) 
        }                                                                           
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
}

결과
1: First at 156 ms from start
2: First at 299 ms from start
3: First at 401 ms from start
3: Second at 901 ms from start

```
