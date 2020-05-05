---
layout: post
title:  "Logstash에서 JDBC Input 사용"
date:   2019-02-16
categories: elasticsearch
---

Logstash에서 JDBC Input를 사용해보도록 하자. 먼저 해당 플러그인을 인스톨한다.

```bash 
/usr/share/logstash/bin/logstash-plugin install logstash-input-jdbc
```

conf 파일을 만들어서 아래와 같이 설정하면 된다. 

```
input {
    jdbc {        
        jdbc_driver_library => "/etc/logstash/mysql-connector-java-8.0.13.jar"
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        jdbc_connection_string => "jdbc:mysql://remoteIP:3306/ykkim?serverTimezone=Asia/Seoul"
        jdbc_user => "user"
        jdbc_password => "1234"
        #실행빈도 (1분마다)
        schedule => "*/1 * * * *"
        #쿼리 매개변수 
        parameters => {"age" => "15"}
        #SQL 문
        statement => "SELECT * FROM users WHERE age >= :age"
    }
}

filter {

}

output {
    stdout {
        codec => rubydebug
    }
}
```

물론 output에도 jdbc를 같은 식으로 사용할 수 있다. 