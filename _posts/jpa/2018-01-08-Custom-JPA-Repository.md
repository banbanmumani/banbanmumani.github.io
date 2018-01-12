---
title: Custom JPA Repository
layout: post
category: jpa
---

### 에러

{% highlight java %}
Caused by: org.springframework.data.mapping.PropertyReferenceException: No property
{% endhighlight %}

GoodRepository

GoodRepositoryCustom

GoodRepositoryCustomImpl

구조로 가져갔을경우 Custom에서 정의한 method를 구현한 곳을 찾지 못해 발생하는 이슈.

### 실수의 원인
커스텀 Repository를 생성 후 구현하려면 RepositoryCustom을 implementation 해야한다고 생각했다.

### 해결
구조적으로 Repository가 JPARepository와 RepositoryCustom을 상속 받고 있기 때문에 RepositoryCustomImpl이 아닌 RepositoryImpl이 되어야 한다.

커스텀 Repository를 사용하려면

GoodRepository

GoodRepositoryCustom

GoodRepositoryImpl

로 구성해야 한다.
