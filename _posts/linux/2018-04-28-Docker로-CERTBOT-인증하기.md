---
title: Docker로 certbot 인증 받기
layout: post
category: Linux
---
### 배경
네이버 톡톡과 챗봇의 연동을 하기 위해 인터페이스 제작을 했다. 그런데 네이버 톡톡은 https 통신을 하기 때문에 SSL 인증서를 발급 받아야한다. 그래서 api용 도메인을 생성하고 SSL 인증서를 letsencrypt 에서 발급받기로 했다.


### 인증받기
letsencrypt 를 이용해서 인증서를 발급받을때 [certbot을 Docker 를 이용하기 위한 명령어](https://certbot.eff.org/docs/install.html#docker-user)는 다음과 같이 소개하고 있다.

{% highlight bash %}
sudo docker run -it --rm --name certbot \
-v "/etc/letsencrypt:/etc/letsencrypt" \
-v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
certbot/certbot certonly
{% endhighlight %}

이 방법을 쓰게 되면 docker 컨테이너 안에서 certbot이 실행되고 있기 때문에 도메인의 소유권을 확인하기 위한 테스트를 진행할때 호스트 서버의 webroot를 찾지 못한다.

{% highlight bash %}
sudo docker run -it --rm --name certbot \
-v "/etc/letsencrypt:/etc/letsencrypt" \
-v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
-v "/var/www/html:/var/www/html" \
certbot/certbot certonly
{% endhighlight %}

이렇게 호스트의 webroot도 연결해줘야 certbot이 테스트를 수행할 수 있다. (각자 사용하고 있는 웹서버의 웹루트 경로를 적어주면 된다.)
