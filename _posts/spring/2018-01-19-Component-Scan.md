---
title: Component Scan가 어떻게 클래스를 찾을까?
layout: post
category: spring
---
### 계기
스터디 할겸 CLI에서 사용할 Sping과 같은 프레임워크를 만들고 있었다. 그러던중 Controller와 같이 라우팅해주는 클래스를 동적으로 등록해야 할 필요성이 생겼다. 동적으로 실행시키기 위해 Reflection을 사용하는데 이때는 정확한 클래스 이름이 있어야만 동적 할당이 된다. 'com.dinky.todo.TodoController' 와 같이 패키지명+클래스명의 풀네임이 필요하다. 그렇기 때문에 Spring에서 Component-Scan과 같이 Bean을 찾아 등록시켜야 했다.

그러기 위해 어찌할지 고민해봤다. 처음 생각엔 Class정보를 바로 가져오면 될것 같았다. 하지만 java는 runtime시 ClassLoader의 Vector<Class>에 접근하지 못한다. 그래서 다음과 같이 접근을 생각했으나

1. BasePackage를 설정
2. BasePackage를 재귀탐색하며 class 목록을 저장
3. 저장한 class들의 Naming, 또는 Annotation을 확인하여 등록

Reflection의 Package는 Class목록을 제공하진 않았다. 남은 방법은 classpath에서 .class 파일을 찾아서 읽어야 할듯 했다.

{% highlight java %}
ClassLoader loader = Thread.currentThread().getContextClassLoader();

String basePackage = "com.dinky.todo";

Enumeration<URL> resources = loader.getResources(basePackage.replace(".", "/"));

List<File> dirs = new ArrayList();

while (resources.hasMoreElements()) {
    URL url = resources.nextElement();
    dirs.add(new File(url.getFile()));
}
{% endhighlight %}

이 방법을 사용하면 .class 파일을 가져올 수 있다.

목록이 디렉토리일 경우는 다시 재귀적 탐색을 통해 모든 .class파일을 확보 한 후 파일을 읽어서 내용을 확인하거나, name으로 대상을 등록하면 될듯 하다.

### 스프링은 어떻게 할까?
왠지 스프링은 더 우아한 방법을 쓰지 않을까?

빈을 검색하고 등록하는 시점에 break point를 찍어서 하나씩 찾아 들어간 결과 다음과 같았다.

![콜스택]({{ site.url }}/assets/spring/component-scan-call-stack.png)

{% highlight java %}
public Set<BeanDefinition> findCandidateComponents(String basePackage) {
  Set<BeanDefinition> candidates = new LinkedHashSet<BeanDefinition>();
  try {
    String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
        resolveBasePackage(basePackage) + '/' + this.resourcePattern;
    Resource[] resources = this.resourcePatternResolver.getResources(packageSearchPath);
    boolean traceEnabled = logger.isTraceEnabled();
    boolean debugEnabled = logger.isDebugEnabled();
    for (Resource resource : resources) {
      if (traceEnabled) {
        logger.trace("Scanning " + resource);
      }
      if (resource.isReadable()) {
        try {
          MetadataReader metadataReader = this.metadataReaderFactory.getMetadataReader(resource);
          if (isCandidateComponent(metadataReader)) {
            ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
            sbd.setResource(resource);
            sbd.setSource(resource);
            if (isCandidateComponent(sbd)) {
              if (debugEnabled) {
                logger.debug("Identified candidate component class: " + resource);
              }
              candidates.add(sbd);
            }
            else {
              if (debugEnabled) {
                logger.debug("Ignored because not a concrete top-level class: " + resource);
              }
            }
          }
          else {
            if (traceEnabled) {
              logger.trace("Ignored because not matching any filter: " + resource);
            }
          }
        }
        catch (Throwable ex) {
          throw new BeanDefinitionStoreException(
              "Failed to read candidate component class: " + resource, ex);
        }
      }
      else {
        if (traceEnabled) {
          logger.trace("Ignored because not readable: " + resource);
        }
      }
    }
  }
  catch (IOException ex) {
    throw new BeanDefinitionStoreException("I/O failure during classpath scanning", ex);
  }
  return candidates;
}
{% endhighlight %}

스프링은 'classpath:* 패키지명칭'으로 접근해서 가져온 후 분석하는 식으로 클레스 정보를 읽어온다.

component-scan이 궁금해서 소스를 열어봤는데 오히려 logging하는 방법을 깨닫아서 기분이 좋았다. 기존에는 레벨별 로그 남기는것에 대해 고민하지 않았는데 logger의 상태를 가져온 후 레벨에 맞게 정보를 출력하는 부분이 인상깊으면서 비지니스로직과 로그 로직이 혼재되서 쉽게 파악이 안되는 아쉬움이 있다. 이를 어찌 해결할지, 이것이 최선인지는 좀더 생각해봐야 겠다.

### 언제나 선배님들이 먼저 만드셨지!
[Reflections](https://github.com/ronmamo/reflections), [fast-classpath-scanner](https://github.com/lukehutch/fast-classpath-scanner)과 같은 다양한 라이브러리를 활용하면 손쉽게 필요한 클래스, 메소드를 찾을 수 있다.
