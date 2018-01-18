---
title: Centos 6.9 Minimal VMWARE 설치 오류
layout: post
category: Linux
---
### 설치시 멈춤 발생
{% highlight bash %}
/etc/rc5.d/S99local: line 25: eject: command not found
/etc/rc5.d/S99local: line 25: eject: command not found
{% endhighlight %}

### 해결
Runlevels 변경으로 해결

VMWARE에서 restart 후 "Press any key" 가 나오면 a를 통해서 커널변수를 추가해준다. 이때 'a 3'을 추가 입력해서 런레벨을 변경하는 것으로 해결함.

[공식사이트](https://centoshelp.org/resources/post-install-options/changing-runlevels/) 참조
