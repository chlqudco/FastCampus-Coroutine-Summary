- 깊이 사용하기 위해 어떤 형태로 구현되어 있는지 이해하는게 좋음

---

- 비동기 처리의 역사?
	- 역사가 아닌거 같은데;
	
- Monad
	- 1950년 후반 로제 고드망이 개념을 만듬
	- 파라미터 이름임
	- monad: (T) -> Box<U>
	- 값을 받아서 박스를 반환하는 함수
	- 대부분의 flatMap(bind, chain, >>=) 함수의 인자는 모나드
	- Monad는 flatMap의 파라미터 이름
	- 뭔소리지?

- Functor
	- Monad의 사촌
	- 펑터(functor)는  값을 받아서 값을 반환하는 함수
	- fun <U> map(functor: (T) -> U): Box<U>
	- 모나드와의 가장 큰 차이는 박스가 아닌 값을 바로 리턴한다는 점
	- 대부분의 map 함수의 인자는 펑터

- 모나드 단점
	- 함수(모나드, 펑터)와 박스를 조합해 프로그래밍하는 스타일이 어렵습니다.
	- Haskell 공부하면 다들 모나드에서 포기할 정도로 모나드는 악명 높음
	- map은 동기, flatMap은 비동기로 된 이상한 구현이 넘침. (예: RxJava)
	- 순수 모나드는 아니지만 RxJava의 성공이 어떻게 보면 놀랍습니다.

- Continuation Passing Style (1975)
	- 순차적 코드는 비동기 처리가 어려움
	- 따라서 Continuation를 생각함
	- (넓은 의미) 다음에 수행해야 할 내용을 의미함
	- (좁은 의미) 다음에 수행해야 할 함수
	- 지속적으로 인자를 전달하면서 코드수행을 하는 스타일
		- 콜백이랑 똑같이 보임
		- 그러나 Continuation는 last call임
	- 따라서 CPS는 끝없는 들여쓰기 때문에 이해하기 매우 어려워짐

```

inChannel.read(buf) { bytesRead ->
    process(buf, bytesRead)
    
    outChannel.write(buf) {
        outFile.close()          
    }
}


```

- Coroutine (1958)
	- 멜빈 콘웨이가 처음 사용
	- Direct Style같은 Straight-forward한 프로그래밍
	- 백준 푸는것 마냥 순서대로 짰는데 알아서 멈추고 복구하고 비동기 적으로 처리함

- Kotlin Coroutine의 특이점
	- 피호출 함수, 람다에만 suspend를 표시

- Coroutine의 지배적인 모델은 C#이 설립한 async/await 모델
	- 중단 가능한 함수는 모두 async로 시작.
	- 반환 값은 Task<String>이라 await가 반드시 필요.
	- 대부분의 언어는 async/await 모델


- 뭐야 왜이리 복잡해?

