---
layout: post
title:  "Spring에서 AOP를 사용해보자."
date:   2014-10-01
categories: java
---

한 어플리케이션의 여러부분에 걸쳐 있는 기능을 가리켜 `횡단관심사(cross-cutting concerns)`라 한다.
주로 로깅이나 세션처리등이 있다.

이런 로깅이나 세션처리작업은 각 모듈에서 반복적으로 사용되고 있으므로 스프링에서는 이러한 것을 `AOP(Aspect-Oriented Programming)`를 통해 처리하고 있다.
따라서 `AOP`의 주된 목적은 횡단관심사의 모듈화에 있다. 즉, 여러모듈에 걸쳐 사용하는 로깅처리나 세션처리등을 별도의 모듈로 분리해서 처리하는 것이다.
이렇게 되면 모든 소스에 걸쳐 표시된 로깅(혹은 세션처리)등을 한군데로 관리함으로써
관리하기가 쉬어지고, 아울러 소스도 깔금해진다.  

먼저 `AOP`가 적용안된 로직을 보자

```java
@RequestMapping(value="/select", method=RequestMethod.POST)
@ResponseBody    
public ResultJSON selectHellUser(HttpServletRequest request, @ModelAttribute HelloUser helloUser) {        
    ResultJSON resultJSON = new ResultJSON();
    try {
        int total = helloUserService.selectTotalCountHelloUser(helloUser);
        List<HelloUser> helloUserList = helloUserService.selectHelloUser(helloUser);
        resultJSON.setTotal(total);
        resultJSON.setItems(helloUserList);        
        resultJSON.setSuccess(true);    
    } catch (Exception e) {
        resultJSON.setSuccess(false);
        resultJSON.setMsg(e.toString());
        logger.error(e);
    }    
    return resultJSON;        
}
```

`selectHellUser` 메소드를 보면 로직 처리시에  `try~catch` 구문으로 오류를 처리하고 로깅처리를 하는 부분이 있다 , 문제는 이런 메소드가 많을 수록 `try~catch` 구문도 많아지는 것이다. 여기에 만약 세션처리부분도 있다면 덩달아 코드가 길어지게 된다.  

다음은 다른 메소드에 `AOP`를 적용한 다음에 처리한 소스이다.

```java
 /**
 * yboard리스트: 파라미터를 json형태로 받고 결과를 json형태로 출력
 *
 * @param yboard
 * @return
 */
@RequestMapping(value = "/select", method = RequestMethod.POST)
@ResponseBody
public ResultJSON selectYboard(@RequestBody YboardSearch yboardSearch) {
    ResultJSON resultJSON = new ResultJSON();
    int totalCount = yboardService.selectTotalCountYboard(yboardSearch);
    List<Yboard> yboardList = yboardService.selectYboard(yboardSearch);
    resultJSON.setTotal(totalCount);
    resultJSON.setItems(yboardList);
    resultJSON.setSuccess(true);
    return resultJSON;
}
```

이전 메소드와 달리 `selectYboard 메소드`는  `try~catch` 구문이 없다.
그렇다고 해서 에러 처리가 안된 것이 아니다. 에러처리는 물론 그에 따른 로깅처리되 되어 있는 상태이다. 다만 그 처리를 `AOP`에서 처리한다는 것이다.
Spring에서 `AOP`를 사용하기 위해서는 AOP에 대한 몇가지 용어에 대한 개념을 알고 있어야 한다.

>어드바이스(Advice): 애스팩트(AOP)가 해야 할 작업.  

어드바이스는 애스팩트가 무엇을 언제 할지를  정한다. 즉 메소드를 호출 이전인지, 이후인지, 모두인지등을 정한다.

- before (이전) :  메소드가 호출되기 전에 수행한다.
- aftert (이후): 결과에 상관없이 메소드가 호출된 다음 수행된다.
- after-returning (반환 이후) : 메소드가 성공적으로 처리된 다음 수행한다.
- after-throwing (예외발생이후 ) : 메소드가 실패하여 예외발생이후에 수행한다.
- around(주위) : 메소드를 감싸서 호출 전후에 수행한다.
- Joinpoint : 어드바이스를 적용할 수 있는 곳, 즉 어플리케이션 실행에 애스팩트를 끼워 넣을 수 있는 지점을 말한다.
- Pointcut : 애스팩트가 어드바이스할 조인포인트의 영역을 좁히는 일을 한다.

어드바이스는 애스팩트가 무엇을 언제 할지를 정의한다면 포인트컷은 어디서 할지를 정의한다.


`AOP`를 Spring에서 사용하기 위해서는 `application.xml`에 `<aop:aspectj-autoproxy/>` 를 추가한다.
그리고 나서 AOP를 처리할 클래스(애스팩트)를 만든다.  

아래는 샘플 소스이다.

```java
@Component
@Aspect
public class YboardAOP extends YboardLogger {

    /**
     * Control에 있는 메소드를 AOP한다.
     *
     * @param joinPoint
     * @return
     */     
    // * 만약 com.yk안에 여러개의 패키지가 있을 경우 || 으로 처리한다. (com.yk.*.*.*.*)
    // @Around("execution(public * com.yk.*.*.*Controller.*(..)) || execution(public * com.yk.*.*.*.*Controller.*(..))")
    @Around("execution(public ResultJSON com.yk.yboard.control.*Controller.*(..))")
    public ResultJSON coverAround(ProceedingJoinPoint joinPoint) {
        ResultJSON resultJSON = new ResultJSON();
        try {
            // 해당 메소드 호출
            resultJSON = (ResultJSON) joinPoint.proceed();
        } catch (Throwable e) {
            resultJSON.setSuccess(false);
            resultJSON.setMsg(e.toString());
            logger.error("[" + joinPoint.toString() + "]*" + e);
        }
        return resultJSON;
    }
}
```

`YboardAOP`를 만들고 나서, `@Aspect`로 지정한 후에 `coverAround`라는 around형 어드바이스를 만든다.
이 메소드에서 로깅처리 및 세션처리등을 하면 향후에 모든 클래스(주로 controller)에서 이 메소드를 통해서 메소드 처리가 진행된다.
또한 around로 지정되어 있기 때문에 메소드의 호출전후에 적용된다. Around를 적용할 클래스를 지정한다.

>@Around("execution(public ResultJSON com.yk.yboard.control.*Controller.*(..)

즉, 위에 내용은 public 메소드이면서 리턴값이 ResultJSON  이고, 경로는 con.yk.yboard.control안에 클래스명이 *Controller로 끝나는 클래스의 모든 메소드에  around형의 `aop`를 적용하다는 뜻이다. 따라서 `selectYboard`는 바로 이 `coverAround`안에서 실행되는 셈이다.

>joinPoint.proceed(); -> 바로 이부분에서 호출이 되어진다.

이렇게 처리하면 모든 메소드에서 `try~catch`구문의 소스가 사라지고, 로깅 혹은 세션등이 한군데서 관리되어지므로 소스는 간결해지고, 관리는 수월해지게 된다.  

- 참조서적 : **[Spring in Action] -그레이그 월즈지음, 홍영표옯김 (출판사 :Jpub)**
