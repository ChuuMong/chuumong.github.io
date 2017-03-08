---
layout: post
category : Android
title : "[Project] Android Project 구성 - Version Name과 Code"
description : ""
tags : [Android, Project]
---

{% include JB/setup %}

> *이 문서는 [Configuring Android Project — Version Name & Code](https://medium.com/@dmytrodanylyk/configuring-android-project-version-name-code-b168952f3323#.vu3hjasc7)를 번역한 문서입니다.*

## Version Name & Code

개발자는 일반적으로 android versionName 및 versionCode에 대해 하드코딩 된 값을 사용합니다.

```
defaultConfig {
    ...
    versionCode 1
    versionName "1.0.0"
}
```

이러한 방식에는 몇가지 단점이 있습니다.

- 특정 버전을 나타내는 커밋을 알 수 없습니다.
- versionCode를 증가시키고 versionName을 변경 할 때마다 build.gradle를 수정해야합니다.

git을 소스 제어 시스템으로 사용한다면 Android versionCode alc versionName을 생성하는데 도움이 될 수 있습니다. git 태그를 사용하여 새 버전의 출시를 나타내는 것이 일반적입니다.

![](https://cdn-images-1.medium.com/max/800/1*cf1L-J-hEr8seSKLn9uHwQ.png)

### Version Name

versionName에 대해서는 [git describe](https://git-scm.com/docs/git-describe) 명령을 사용합니다.

1. 이 명령은 커밋에서 도달 할 수 있는 가장 최근의 태그를 찾습니다.

2. 태그가 커밋을 가리키면 태그만 표시됩니다.

3. 그렇지 않으면 태그 이름 뒤에 태그가 붙은 객체의 맨 위에 있는 추가 커밋수와 가장 최근 커밋의 약식 객체 이름을 붙입니다.

**Example (1-2)**

![](https://cdn-images-1.medium.com/max/800/1*JWBvoMJo89W_RQYqPU3Utw.png)

1. 태그 1.0으로 특정 커밋을 표시
2. 이 커밋을 check out
3. `git describe -tags`를 호출
4. 출력 : `1.0`

`HEAD` 커밋에서 git이 몇 가지 태그를 사용하여 설명하는 것을 볼 수 있듯이 이 태그를 출력합니다.

**Example (1-3)**

![](https://cdn-images-1.medium.com/max/800/1*is7JdLND-m0V2VI_19fAsQ.png)

1. 태그 1.0으로 커밋 표시
2. 커밋 2 개 더 추가
3. `git describe-tags`를 호출
4. 출력 : `1.0-2-gdca226a`

![](https://cdn-images-1.medium.com/max/800/1*JerRUWHIiqTh71UfwbI6-w.png)

1. TAG 이름
2. 커밋 된 태그 이후 커밋 수
3. 짧은 형식의 커밋 해시 "dca226a"와 "g" 접두어 ("git"을 나타냄)

![](https://cdn-images-1.medium.com/max/800/1*HhFvJXnXdAAiXqDF_Epzxg.png)

### Version Code

versionCode의 경우 총 태그 수를 사용할 수 있습니다. 모든 자식 태그는 일부 버전을 나타 내므로 다음 버전의 versionCode는 항상 이전 버전보다 큽니다.

![](https://cdn-images-1.medium.com/max/800/1*aDkEKS33PqNoy5YzarWr4A.png)

위의 예에서는 3 개의 태그가 있습니다. 이 값은 versionCode에 사용됩니다.

그러나 모든 중간 버전에 대해 git 태그를 만들지 않으므로 dev 빌드의 경우 HEAD 커밋의 타임 스탬프를 사용할 수 있습니다.

![](https://cdn-images-1.medium.com/max/800/1*EwUZBO8vMcBX215VNh0VMQ.png)

위의 예에서 HEAD 커밋의 타임 스탬프는 1484407970 (UNIX 1970년 1월 1일 00:00:00 UTC 이후의 초)과 같습니다. 이 값이 versionCode에 사용됩니다. 사람이 읽을 수있는 날짜로 변환하려면 currentmillis.com 사이트를 사용하십시오. 우리의 경우 그것은 Sat Jan 14 2017 15:32:50 UTC입니다.

### Groovy에서 git 사용

git로 작업하려면 grgit이라는 라이브러리를 사용하는 것이 좋습니다. 다음 내용이 포함 된 script-git-version.gradle 파일을 만듭니다.

```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'org.ajoberstar:grgit:1.5.0'
    }
}

import org.ajoberstar.grgit.Grgit

ext {
    git = Grgit.open(currentDir: projectDir)
    gitVersionName = git.describe()
    gitVersionCode = git.tag.list().size()
    gitVersionCodeTime = git.head().time
}

task printVersion() {
    println("Version Name: $gitVersionName")
    println("Version Code: $gitVersionCode")
    println("Version Code Time: $gitVersionCodeTime")
}
```

build.gradle 파일에 적용하십시오.

```
apply plugin: 'com.android.application'
apply from: "$project.rootDir/tools/script-git-version.gradle"
```

버전 이름과 코드가 올바르게 생성되었는지 확인하려면 `./gradlew printVersion`을 사용하면 결과를 얻을 수 있습니다.

```
Version Name: 1.0-2-gdca226a
Version Code: 2
Version Code Time: 1484407970
```

마지막으로 build.gradle 파일에서 gitVersionName, gitVersionCode 및 gitVersionCodeTime 변수를 사용하십시오.

```
productFlavors {
    dev {
        versionCode gitVersionCodeTime
        versionName gitVersionName
    }

    prod {
        versionCode gitVersionCode
        versionName gitVersionName
    }
}
```

프로젝트를 실행하고 앱 버전을 확인하십시오.

![](https://cdn-images-1.medium.com/max/800/1*zIjpTE3Ldtrj_tZqxsgnvw.png)

이러한 접근법의 이점은
- build.gradle 파일을 수정할 필요가 없습니다. - versionCode 및 versionName이 자동으로 생성됩니다.
- 빌드가 만들어진 커밋을 쉽게 알 수 있습니다.

> 참고 : 버전 이름은 더 많이 실험해볼 수 있습니다. (분기 이름, 타임 스탬프 등을 포함)
