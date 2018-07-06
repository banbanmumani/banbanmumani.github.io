---
title: Bash Shell 상대 경로 삽질
layout: post
category: etc
---
## 삽질의 배경
C 개발자가 만든 모듈을 웹에 연동해야 해서 데몬으로 만들어서
* socket이나 MQ로 하자고 했으나 C 개발자분이 socket이나 MQ가 익숙하지 않다고 하셔서
* Bash Shell 만들면 그걸 java로 실행
으로 방향으로 결정했다.

그런데 실행이 안된다..

## 삽질 환경
jar로 패키징한 웹서버와 C_Module, 둘을 연결할 run.sh 쉘스크립트가 오늘의 주인공이다. 환경은 다음과 같다

Web-Server.jar
* 경로
  * /data/www/Web-Server.jar

run.sh
* 경로
  * /data/bin/run.sh

  * 내용
{% highlight bash %}
#!/bin/bash
./C_Module -input $1 -output $2
{% endhighlight %}


C_Module
* 경로
  * /data/bin/C_Module

## 해결을 위한 몸부림
이슈
* shell 안에 c 모듈이 실행 안된다.

가정
* 쉘안에서 선언한 다른 명령어가 실행이 안되는 것이다

테스트
1. 인텔리제이로 프로젝트 루트에 쉘파일을 만들어서 outer shell 안에 inner shell 을 호출하는 파일을 만들어서 테스트 했으나 잘 실행됨

2. 서버에서 라인마다 '&& echo (라인넘버) proc' 를 하며 실행 라인을 따라가봤는데 inner 호출 부분만 실행이 안되고 다음라인을 실행시킴

3. (해결) run.sh의 내용을 ./C_Module 을 cd /data/bin/ && ./C_Module 과 같이 변경함

# 결론
쉘을 실행하는 위치가 쉘의 위치와 다른 경우 내부에 선언된 상대경로가 문제가 발생하므로 쉘이 위치한 곳으로 이동 후 실행 하거나 쉘 내부에 선언된 부분에 이동하는 명령어를 추가하자.


# 이번에 배운점
1. java 로 process 실행하기
{% highlight java %}
Process p = Runtime.getRuntime().exec({"/data/bin/run.sh", "/data/input/input.txt", "/data/output/output.txt"});
{% endhighlight %}
