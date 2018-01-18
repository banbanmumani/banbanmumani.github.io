---
title: Notability 마이너 만들기(1)
layout: post
category: toy project
---
### 계기
notability를 회의시 회의록 작성용도로 사용중인데 회의중에 해야할 일이라고 여겨지는 내용들을 '//todo' 로 작성했다. 이것을 나중에 다시 수집할때 번거롭기 때문에 노트에 '//todo'로 작성된 부분을 수집해서 todo list로 추출하고 싶다.

### Notability가 plugin api를 지원하는가?
notability 공식 홈페이지에는 sharing 과 관련된 부분만 존재하고 외부 API 안내는 없다.

### 구현 방안
메모 파일에서 '// todo'영역을 수집한다. 그러나 일반적으로 osx의 어플리케이션이 파일을 저장하는 위치에 (~/Library/Application Support/<app-name>) 데이터를 찾지 못했다. .app패키지를 열어봐도 파일로 저장하지 않고 db를 이용하는 듯 보여서 다른방법을 찾아보기로 했다.

### 구현 방안 변경
클라우드 서비스에 메모 자동 백업을 시켜놓고 클라우드에 저장된 메모를 불러와서 '// todo'영역을 수집한다

### 클라우드 선정
평소에 iCloud를 사용하고 있어서 시도했으나 Notability의 메모가 저장되는 영역은 Sandbox로 접근할 수 없어서 api제공이 친절한 google drive를 이용하기로 결정했다. 게다가 Notability에서 백업 타이밍은 메모 작성/수정 이후 포커스가 다른 메모로 변경되는 시점에 실시간으로 이루어 지기 때문에 todo 추출도 실시간으로 이루어질 수 있다.

### 구성
크게 google drive에서 데이터를 가져오는 마이너 모듈과 데이터로부터 필요한 내용을 파싱하는 파서모듈로 구현하기로 결정했다.

### 마이너 구현 - api key 얻기
[Api 키를 얻어야 한다](https://console.developers.google.com/flows/enableapi?apiid=drive&pli=1)

### 마이너 구현 - 소스작성
마이너 부분은 [구글 드라이브 클라이언트 샘플](https://github.com/google/google-api-java-client-samples/tree/master/drive-cmdline-sample) 코드를 참고해서 진행했다.

#### 이슈 1 : 특정 클래스를 찾지 못함
{% highlight groovy %}
group 'com.dinky'
version '1.0-SNAPSHOT'

apply plugin: 'java'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
    compile 'com.google.apis:google-api-services-drive:v3-rev86-1.22.0'
}
{% endhighlight%}

build.gradle 을 다음과 같이 작성했는데 아래의 두 클래스를 찾지 못했다.

{% highlight java %}
import com.google.api.client.extensions.java6.auth.oauth2.AuthorizationCodeInstalledApp;
import com.google.api.client.extensions.jetty.auth.oauth2.LocalServerReceiver;
{% endhighlight %}



{% highlight groovy %}
compile group: 'com.google.oauth-client', name: 'google-oauth-client-java6', version: '1.11.0-beta'
compile group: 'com.google.oauth-client', name: 'google-oauth-client-jetty', version: '1.11.0-beta'
{% endhighlight %}

그래서 위 두줄을 추가해서 개발을 진행했다.

#### 이슈 2 : 파일 목록을 다 가져오지 못함

{% highlight java %}
private Credential authorize() throws Exception {
    GoogleClientSecrets clientSecrets = GoogleClientSecrets.load(JSON_FACTORY
    , new InputStreamReader(Miner.class.getResourceAsStream("/client_secrets.json")));

    GoogleAuthorizationCodeFlow flow = new GoogleAuthorizationCodeFlow.Builder(
            httpTransport, JSON_FACTORY, clientSecrets,
            Collections.singleton(DriveScopes.DRIVE_FILE)).setDataStoreFactory(dataStoreFactory)
            .build();
    return new AuthorizationCodeInstalledApp(flow, new LocalServerReceiver()).authorize("user");
}
{% endhighlight %}

인증하는 코드에서 DriveScopes에 따라 접근할 수 있는 권한이 달라진다. DRIVE_FILE 로 범위를 설정하면 이 애플리케이션이 생성한 파일만 읽고 쓸 수 있고 기존에 드라이브에 있는 파일은 접근할 수 없다. 범위를 DRIVE로 할 경우 전체 구글드라이브 파일에 접근할 수 있다.

더 많은 설명은 [구글드라이브 스코프 설명](https://developers.google.com/drive/v2/web/about-auth#top_of_page)에서 확인할 수 있다.
