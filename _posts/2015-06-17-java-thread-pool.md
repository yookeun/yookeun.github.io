---
layout: post
title:  "자바에서 스레드풀(Thread Pool)관리"
date:   2015-06-17
categories: java
---

자바에서 스레드를 필요한 작업마다 생성하는 것은 크게 자원낭비와 안정성에 문제가 있다.


스레드를 생성하고 제거하는 작업에는 자원이 소모된다. 스레드가 많다면 대부분의 대기중인 상태가 되고, 더더욱 메모리 사용량이 많아지고 JVM의 가비지콜렉터에 많은 부담을 주게된다.
또한 모든 시스템에는 생성할 수 있는 스레드의 개수가 제한되어 있다. 만약 제한된 양을 모두 사용하게 되면 OutOfMemoryError를 보게 될 것이다.

이러한 단점을 해소하기 위해서 스레드 풀(Thread Pool)이라는 것을 이용해서 스레드를 관리할 수가 있다. 이미 자바에서는 `java.util.concurrent.Executors`에서 스레드풀을 제공하고 있다.


```java
public interface Executors {
	void execute(Runnable command);
}
```

`Executors`의 구조는 프로듀서-컨슈머 패턴으로 만들어져 있다.
따라서 스레드가 생성하고 나서 같은 작업을 다시 하는 일이 없다면 스레드가 처리하고 나서 바로 종료처리해도 되지만, 반복적으로 어떠한 작업을 어떠한 시점에서 해야 한다면
매번 스레드를 생성하고 종료하는 로직보다는 해당스레드가 계속 대기중인 상태가 되어 작업할 시점이 왔을 때 처리하는 것이 좋다.

스레드풀을 사용한다면 매번 스레드를 생성하는 것보다 많은 장점이 있다. 즉 이전의 스레드를 재사용할 수 있으므로 시스템자원을 줄일 수 있고, 작업을 요청시 이미 스레드가 대기중인 상태이기때문에서 작업을 실행하는데 딜레이가 발생하지 않는다. `Executors`에서는 설정에 따른 여러가지 스레드풀을 제공하고 있다.

### newFixedThreadPool ###
> 주어진 스레드개수만큼 생성하고, 그 수를 유지한다. 이때 생성된 스레드중 일부가 작업시 종료되었다면 스레드를 다시생성하여 주어진 수를 맞춘다.

### newCachedThreadPool ###
> 처리할 작업의 스레드가 많아지면 그 만큼 스레드를 증가하여 생성한다. 만약 쉬는 스레드가 많다면 스레드를 종료시킨다. 반면 스레드를 제한두지 않기때문에 조심히 사용해야 한다.

### newSingleThreadExecutor ###
> 스레드를 단 하나만 생성한다. 만약 스레드가 비정상적으로 종료되었다면 다시 하나만 생성한다.

### newScheduledThreadPool ###
> 특정시간 이후에 실행되거나 주기적으로 작업을 실행할 수 있다.


다음과 같이 스레드풀을 생성하는 메소드를 만들어서 처리할 수 있다.

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 멀티 스레드풀 실행
 *
 * @param object
 * @param count
 */
public void executeMultiExecutor(Object object, int count) {
	ExecutorService executorService = Executors.newFixedThreadPool(count);  //항상 일정한 스레드 개수를 유지한다.
	for (int i = 0; i < count; i++) {
		executorService.execute((Runnable) object);
	}
}

```

만약, TestThread라는 스레드를 10개 고정시켜서 생성시키고자 한다면 아래과 같이 호출하면 된다.


```java
executeMultiExecutor(new TestThread, 10);
```

물론 TestThread 스레드는 while(true)구문으로 큐를 처리하고 대기하는 기능이 구현되어 있어야 한다.  
그런거없이 무조건 스레드가 유지되지는 않는다.  

`Executors`에서는 newFixedThreadPool, newCachedThreadPool, newSingleThreadExecutor, newScheduledThreadPool메소드가 존재하니 용도에 맞게 사용하면 된다.
