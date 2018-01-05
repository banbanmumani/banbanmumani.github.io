---
title: 맥(osx)에서 oracle DB 사용하기
category: docker
layout: post
---
### 고통의 과거
2년전 국비지원 수업을 들을때 oracle 10g로 sql을 처음 접했다. 난 맥북을 가지고 수업을 듣고 있었고 oracle 10g를 osx에 설치하지 못해서 VMWARE에 DB를 설치해서 포트를 연결해서 썼던 기억이 난다. 하지만 docker를 알게된 지금은 그럴필요가 없다.

### docker 설치
docker를 [다운로드](https://download.docker.com/mac/stable/Docker.dmg) 받자. 설치는 쉽다.

![설치]({{ site.url }}/assets/docker/docker_install.png)

![실행화면]({{ site.url }}/assets/docker/docker_state.png)

도커 설치가 완료된 후 도커실행이 완료되면 oracle을 사용할 준비가 됐다.

### docker에 대해 최소한 알아야 할것
도커는 무상태를 지향한다. 그렇기 때문에 프로그램(컨테이너)을 실행시킨다음 생성된 데이터는 프로그램(컨테이너)을 종료하면 데이터는 저장되지 않고 사라진다. 그렇기 때문에 oracle을 docker로 사용하려면 데이터를 프로그램(컨테이너)외부에 저장해야 한다. 그래서 -v 옵션으로 외부에 데이터 저장할 것 이다.

### oracle 설치
도커로 사용하고 싶은 프로그램이 있다면 구글에 'docker 프로그램이름'을 검색하거나 [도커허브](https://hub.docker.com/)에서 원하는 프로그램을 검색하면 된다. 지금은 [oracle 12c](https://hub.docker.com/r/sath89/oracle-12c/)를 사용하겠다.

{% highlight bash %}
docker run --name oracle12c -d -p 8080:8080 -p 1521:1521 sath89/oracle-12c #실행하지 마세요.
{% endhighlight %}

이렇게 하면 오라클이 실행 될것이다. 다만 위에서 언급한대로 데이터는 컨테이너를 중지하는 순간 사라질것이다. 혹시나 위 명령어를 실행했으면 이미 이미지가 생성되서 동일한 이름이 있기 때문에 아래 명령어로 이미지를 지우거나 새로운 컨테이너 이름을 사용해야 한다.

{% highlight bash %}
docker rm oracle12c
{% endhighlight %}
rm 명령어는 컨테이너를 삭제하는 명령어다.

{% highlight bash %}
docker run --name oracle12c -d -p 8080:8080 -p 1521:1521 -v ~/my/oracle/data:/u01/app/oracle sath89/oracle-12c
{% endhighlight %}

이렇게 -v 옵션으로 저장될 위치를 설정해주면 해당 위치에 데이터를 저장하기 때문에 데이터는 보존된다. 위 명령어는 '~/my/oracle/data'라는 경로에 데이터를 저장하겠다는 의미므로 다른 위치에 저장하고 싶으면 해당 문구를 변경하면 된다.

![이미지 다운로드]({{ site.url }}/assets/docker/docker_oracle_pull_image.png)

{% highlight bash %}
docker logs -f oralce12c
{% endhighlight %}
오라클은 초기화가 오래걸리는 편이라 위의 로그출력 명령어로 진행상황을 보면 마음이 편안해 진다.

![로그]({{ site.url }}/assets/docker/docker_oracle_logs.png)

초기화가 완료되면 이제 oracle을 사용할 준비가 끝난다.

### lsof 명령어로 포트확인

{% highlight bash %}
lsof -PiTCP -sTCP:LISTEN
{% endhighlight %}

![포트]({{ site.url }}/assets/docker/docker_oracle_lsof.png)

1521 포트와 웹콘솔인 8080 포트가 열려있는것을 확인할 수 있다.

### oracle12c 사용
#### sqlplus
sqlplus로 접속하려면 다음과 같은 방법으로 접속할 수 있다.

{% highlight bash %}
docker exec -it oracle12c sqlplus
{% endhighlight %}
계정은 system 비밀번호는 oracle 을 입력하면 sqlplus에 접속할 수 있다.

![sqlplus]({{ site.url }}/assets/docker/docker_oracle_sqlplus.png)

#### 웹콘솔

[http://localhost:8080/apex](http://localhost:8080/apex) 로 접속할 수 있다.

workspace: INTERNAL

Username: ADMIN

Password: 0Racle$

![login]({{ site.url }}/assets/docker/docker_oracle_web_console_login.png)

![change]({{ site.url }}/assets/docker/docker_oracle_web_console_change_pw.png)

![main]({{ site.url }}/assets/docker/docker_oracle_web_console_main.png)


#### intellij db툴 이용
![intellij]({{ site.url }}/assets/docker/docker_oracle_connect_test.png)

#### 이슈 1 : 로케일을 인식할 수 없습니다
시에라 이후일때 발생하는 문제로 [로케일변경](https://limsungmook.github.io/2016/11/04/locale-not-recognized-after-upgrade-sierra/)으로 해결할 수 있다.

### 도커 컨테이너 상태보기
{% highlight bash %}
docker ps
{% endhighlight %}

### oracle12c 중지
{% highlight bash %}
docker stop oracle12c
{% endhighlight %}

### oracle12c 컨테이너 삭제
{% highlight bash %}
docker rm oracle12c
{% endhighlight %}

### oracle12c 이미지 삭제
{% highlight bash %}
docker rmi sath89/oracle-12c
{% endhighlight %}
