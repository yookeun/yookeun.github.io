---
layout: post
title:  "우분투(Ubuntu 14.04)에 하둡(Hadoop)을 가상모드로 설치하기"
date:   2015-03-25
categories: java
---

하둡(Hadoop)을 설치하려면 먼저 우분투에 자바가 설치되어 있어야 한다.

오라클 자바7 버전 이상을 설치하도록 한다(가급적 다운받아 설치하는게 좋다)
그리고 나서 설정이 등록되어야 한다.

{% highlight bash %}
vi /etc/profile 열고 아래 경로를 추가하자.

JAVA_HOME=/home/ykkim/java7
PATH:$PATH:$JAVA_HOME/bin

//추가한후에 바로 적용하자.
source /etc/profile
{% endhighlight %}

자. 이제 하둡을 받자. 아래 사이트로 가자 미러링사이트가 훨씬 빠르기 때문이다.

<http://mirror.apache-kr.org/hadoop/common/>
거기서 `hadoop-1.2.1` 로 가서 `hadoop-1.2.1.tar.gz` 를 다운받는다.

자신의 홈디렉토리로 옮긴다음에 압축을 풀고 심볼링링크를 만들자.

{% highlight bash %}
tar xvfz hadoop-1.2.1.tar.gz
ln -s hadoop-1.2.1 hadoop
{% endhighlight %}

`hadoop/conf` 디렉토리로 이동한다. 자 이제부터 설정파일들을 수정하자.

### 1. hadoop-env.sh

9번째 라인쯤에 보면 JAVA_HOME 설정하는 부분이 있다. 거기다 바로 추가하자.

{% highlight bash %}
export JAVA_HOME=/home/ykkim/java
{% endhighlight %}

### 2. core-site.xml

{% highlight xml %}
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<!-- Put site-specific property overrides in this file. -->

<configuration>
 <property>
   <name>fs.default.name</name>
   <value>hdfs://localhost:9000</value>
 </property>
 <property>
   <name>hadoop.tmp.dir</name>
   <value>/home/ykkim/hadoop/hadoop-data/</value>
 </property>
</configuration>
{% endhighlight %}

### 3. hdfs-site.xml

{% highlight xml %}
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!-- Put site-specific property overrides in this file. -->
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
</configuration>
{% endhighlight %}

### 4. mapred-site.xml

{% highlight xml %}
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!-- Put site-specific property overrides in this file. -->
<configuration>
  <property>
    <name>mapped.job.tracker</name>
    <value>localhost:9001</value>
  </property>
</configuration>
{% endhighlight %}

자. 이제 하둡을 실행해보자

{% highlight bash %}
/bin/hadoop namenode -format
{% endhighlight %}

그러면 아래처럼 무언가 엄청난 것이 수행된다.

{% highlight bash %}
15/03/22 17:40:10 INFO namenode.NameNode: STARTUP_MSG:
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = ykserver/127.0.1.1
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 1.2.1
STARTUP_MSG:   build = https://svn.apache.org/repos/asf/hadoop/common/branches/branch-1.2 -r 1503152; compiled by 'mattf' on Mon Jul 22 15:23:09 PDT 2013
STARTUP_MSG:   java = 1.7.0_75
************************************************************/
15/03/22 17:40:10 INFO util.GSet: Computing capacity for map BlocksMap
15/03/22 17:40:10 INFO util.GSet: VM type       = 64-bit
15/03/22 17:40:10 INFO util.GSet: 2.0% max memory = 932184064
15/03/22 17:40:10 INFO util.GSet: capacity      = 2^21 = 2097152 entries
15/03/22 17:40:10 INFO util.GSet: recommended=2097152, actual=2097152
15/03/22 17:40:11 INFO namenode.FSNamesystem: fsOwner=ykkim
15/03/22 17:40:11 INFO namenode.FSNamesystem: supergroup=supergroup
15/03/22 17:40:11 INFO namenode.FSNamesystem: isPermissionEnabled=true
15/03/22 17:40:11 INFO namenode.FSNamesystem: dfs.block.invalidate.limit=100
15/03/22 17:40:11 INFO namenode.FSNamesystem: isAccessTokenEnabled=false accessKeyUpdateInterval=0 min(s), accessTokenLifetime=0 min(s)
15/03/22 17:40:11 INFO namenode.FSEditLog: dfs.namenode.edits.toleration.length = 0
15/03/22 17:40:11 INFO namenode.NameNode: Caching file names occuring more than 10 times
15/03/22 17:40:11 INFO common.Storage: Image file /home/ykkim/hadoop/hadoop-data/dfs/name/current/fsimage of size 111 bytes saved in 0 seconds.
15/03/22 17:40:12 INFO namenode.FSEditLog: closing edit log: position=4, editlog=/home/ykkim/hadoop/hadoop-data/dfs/name/current/edits
15/03/22 17:40:12 INFO namenode.FSEditLog: close success: truncate to 4, editlog=/home/ykkim/hadoop/hadoop-data/dfs/name/current/edits
15/03/22 17:40:12 INFO common.Storage: Storage directory /home/ykkim/hadoop/hadoop-data/dfs/name has been successfully formatted.
15/03/22 17:40:12 INFO namenode.NameNode: SHUTDOWN_MSG:
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at ykserver/127.0.1.1
************************************************************/
{% endhighlight %}

이제 정말 하둡과 관련된 데몬을 실행시킨다. 그런데 `localhost`로 설정되어 있으면 중간에 `localhost`로 계속진행할것인지 물어본다.
`yes` 로 하고 패스워드를 본인계정의 패스워드로 처리하면 된다.

{% highlight bash %}
ykkim@ykserver:~/hadoop$ ./bin/start-all.sh
starting namenode, logging to /home/ykkim/hadoop-1.2.1/libexec/../logs/hadoop-ykkim-namenode-ykserver.out
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ECDSA key fingerprint is 6c:3b:48:d3:42:36:ae:d2:bd:11:c0:32:4d:51:14:03.
Are you sure you want to continue connecting (yes/no)? yes
localhost: Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
ykkim@localhost's password:
localhost: starting datanode, logging to /home/ykkim/hadoop-1.2.1/libexec/../logs/hadoop-ykkim-datanode-ykserver.out
ykkim@localhost's password:
localhost: starting secondarynamenode, logging to /home/ykkim/hadoop-1.2.1/libexec/../logs/hadoop-ykkim-secondarynamenode-ykserver.out
starting jobtracker, logging to /home/ykkim/hadoop-1.2.1/libexec/../logs/hadoop-ykkim-jobtracker-ykserver.out
ykkim@localhost's password:
localhost: starting tasktracker, logging to /home/ykkim/hadoop-1.2.1/libexec/../logs/hadoop-ykkim-tasktracker-ykserver.out
ykkim@ykserver:~/hadoop$
{% endhighlight %}

콘솔에서 확인해보자 jps를 치자.

{% highlight bash %}
jps
{% endhighlight %}
아래와 같이 표시될 것이다.

{% highlight bash %}
NameNode
DataNode
SecondaryNameNode
jps
{% endhighlight %}
등이 표시 될것이다.

이제 최종적으로 화면에서 확인하자.
`http://서버아이피:50070` 으로 접속해서 아래 화면이 나오면 하둡이 드디어 설치완료되고 잘 실행중이라는 뜻이다.

<div style="text-align:center;margin-bottom: 30px;"><img src="/assets/images/hadoop.jpg" style="width:480px"></div>  
