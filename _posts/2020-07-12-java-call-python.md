---
layout: single
title: "Java에서 Python 파일 실행"
date: 2020-07-12
categories: [java]
tags: [java, python]
---

Java에서 Python파일을 호출해서 실행해보도록 하자. 보통 `processBuilder` 등을 사용하는데 여기서는 [Apace Commons Exec](https://commons.apache.org/proper/commons-exec/index.html) 를 사용하도록 한다.

#### Gradle 설정

```groovy
implementation 'org.apache.commons:commons-exec:1.3'
```

#### Python 파일

```python
import sys

def sum(v1,v2):
    result = int(v1) + int(v2)
    print(result)


def main(argv):
    sum(argv[1], argv[2])

if __name__ == "__main__":
    main(sys.argv)
```

간단하게 파라미터를 받아서 합계를 구해준다. 이제 자바에서 호출해보자

```java
package com.call.python;

import org.apache.commons.exec.CommandLine;
import org.apache.commons.exec.DefaultExecutor;
import org.apache.commons.exec.PumpStreamHandler;

import java.io.ByteArrayOutputStream;
import java.io.IOException;

public class CallMain {
    public static void main(String[] args)  {
    	System.out.println("Python Call");
        String[] command = new String[4];
        command[0] = "python";
        //command[1] = "\\workspace\\java-call-python\\src\\main\\resources\\test.py";
        command[1] = "/Users/ykkim/workspace/java-call-python/src/main/resources/test.py";
        command[2] = "10";
        command[3] = "20";
        try {
            execPython(command);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void execPython(String[] command) throws IOException, InterruptedException {
        CommandLine commandLine = CommandLine.parse(command[0]);
        for (int i = 1, n = command.length; i < n; i++) {
            commandLine.addArgument(command[i]);
        }

        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
        PumpStreamHandler pumpStreamHandler = new PumpStreamHandler(outputStream);
        DefaultExecutor executor = new DefaultExecutor();
        executor.setStreamHandler(pumpStreamHandler);
        int result = executor.execute(commandLine);
        System.out.println("result: " + result);
        System.out.println("output: " + outputStream.toString());

    }

}

```

CommandLine를 통해 파라미터등을 처리한다. ByteArrayOutputStream를 통해서 실행 결과값을 받는데 사용한다.

### 실행결과

```bash
> Task :CallMain.main()
Python Call
result: 0
output: 30
```

result가 0으로 되면서 성공적으로 수행되고 output를 통해서 결과를 확인할 수 있다. 이 값을 통해 자바에서 핸들링이 가능하다.
