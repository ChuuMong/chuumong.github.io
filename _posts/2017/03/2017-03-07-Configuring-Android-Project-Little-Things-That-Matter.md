---
layout: post
category : Android
title : "[Project] Android Project 구성 - 중요한 사항"
description : ""
tags : [Android, Project]
---

{% include JB/setup %}

> *이 문서는 [Configuring Android Project — Little Things That Matter](https://medium.com/@dmytrodanylyk/configuring-android-project-little-things-that-matter-d6a9d34c1ce0#.nksj5i62k)의 번역본입니다.*

## Android Project 구성 - 중요한 사항

### gitignore

Android Studio에서 새 프로젝트를 만들 때 gitignore 파일을 제공받지만 모든 규칙이 포함되어 있지 않습니다.

gitignore 파일을 신속하게 생성하고 다운로드하려면 [gitignore.io](https://www.gitignore.io/) 사이트를 사용하는 것이 좋습니다. Android, Intellij와 같은 키워드를 입력하고 generate 버튼을 클릭하세요.

![](https://cdn-images-1.medium.com/max/800/1*BetlRiqYaumLBBFIMrTdkQ.png)

> 템플릿 프로젝트에서 [gitignore](https://github.com/dmytrodanylyk/template/blob/master/.gitignore) 파일을 확인해보세요.

### tools folder

프로젝트에 관련된 일부 타사 *스크립트*, *rulesets* 또는 *기타 파일*을 루트 디렉토리에 놓지 않으면 혼란이 생길 수 있습니다. (특히 Project View를 Android View가 아닌 Project View를 사용하는 경우)

이 럴 떄는 폴더(도구 폴더)를 만들어서 이 폴더에 모든 파일을 저장하세요.

![](https://cdn-images-1.medium.com/max/800/1*4x6EGq81jD673WGhRHF79A.png)

보통 사용자 정의 gradle 파일, [proguard](https://developer.android.com/studio/build/shrink-code.html)에 대한 규칙 및 [pmd](https://pmd.github.io/), [findbugs](http://findbugs.sourceforge.net/), [lint](https://developer.android.com/studio/write/lint.html)와 같은 정적 코드 분석 도구를 넣습니다.

템플릿 프로젝트의 [tools](https://github.com/dmytrodanylyk/template/tree/master/tools) 폴더를 확인하십시오.

### flavors

flavors는 다른 설정으로 빌드를 만드는데 사용됩니다. 대부분의 경우 나는 두가지 종류의 dev와 prod를 설정합니다.


- applicationId
- versionCode / versionName
- server endpoints
- google services keys
- …

```
productFlavors {
    dev {
        signingConfig signingConfigs.debug
        versionCode gitVersionCodeTime
        versionName gitVersionName
    }

    prod {
        signingConfig signingConfigs.release
        versionCode gitVersionCode
        versionName gitVersionName
    }
}
```

> 템플릿 프로젝트의 [productFlavors](https://github.com/dmytrodanylyk/template/blob/master/app/build.gradle#L33)를 확인해보세요.

### keystore

Keystore는 어플리케이션에 서명하는데 사용되는 개인키가 들어가있는 바이너리 파일입니다.

IDE에서 프로젝트를 실행하거나 디버깅 할 때 Android Studio는 Android SDK 도구에서 생성한 디버그 인증서로 APK에 자동으로 서명합니다.

로컬 디버그 keystore를 사용 할 때 몇가지 문제가 있습니다.

- 만료일이 365일
- 여러 디바이스에서 어플리케이션을 설치하려면 제거해야합니다.
- Google 서비스에는 keystore SHA-1 지문이 필요합니다.

그래서 보통 디버그 keystore를 생성하고 버전 제어 시스템에 커밋합니다.

```
signingConfigs {
   debug {
       keyAlias 'androiddebugkey'
       keyPassword 'android'
       storePassword 'android'
       storeFile file('../keystore/debug.keystore')
   }
   release {
       ...
   }
}
```

> 템플릿 프로젝트의 signingConfigs를 확인해보세요.

### Proguard

Android Proguard는 세 가지 용도로 사용됩니다.

- 사용하지 않는 코드 축소(64k 메소드 한도에 도움됩니다.)
- 코드 및 APK 최적화
- 코드 난독화(APK를 리버싱하기 어렵습니다.)

문제는 난독화 및 코드 최적화가 컴파일 시간을 크게 늘리고 디버깅 하기 어렵게 만드는 것입니다.
그렇기 때문에 릴리즈 및 디버그 빌드에 대해 다른 Proguard 규칙을 사용하는것이 좋습니다.

```
buildTypes {
    release {
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android.txt'),
                "$project.rootDir/tools/rules-proguard.pro"
        signingConfig signingConfigs.release
    }
    debug {
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android.txt'),
                "$project.rootDir/tools/rules-proguard-debug.pro"
        signingConfig signingConfigs.debug
    }
}
```

디버그 빌드에 대한 Proguard 규칙에는 Proguard가 경고를 무시하고 코드 난독화 및 최적화를 수행하지 못하게 하도록 다음 코드가 있어야합니다.

```
-dontobfuscate
-dontoptimize
-ignorewarnings
```

릴리즈 버전의 경우 거의 모든 라이브러리에 고유한 규칙이 있으므로 Proguard 규칙을 설정하는것이 어려울 것입니다. 다행히도 주요 라이브러리에 대한 Proguard 규칙을 가지고 있는 [android-proguard-snippets](https://github.com/krschultz/android-proguard-snippets)라는 오픈 소스 저장소가 있습니다.

```
# Add project specific ProGuard rules here.

# Remove logs
-assumenosideeffects class android.util.Log {
   public static boolean isLoggable(java.lang.String, int);
   public static int v(...);
   public static int i(...);
   public static int w(...);
   public static int d(...);
   public static int e(...);
}

# Proguard configurations for common Android libraries:
# https://github.com/krschultz/android-proguard-snippets
```

> 템플릿 프로젝트에서 [rules-proguard.pro](https://github.com/dmytrodanylyk/template/blob/master/tools/rules-proguard.pro)와 [rules-proguard-debug.pro](https://github.com/dmytrodanylyk/template/blob/master/tools/rules-proguard-debug.pro)를 확인해보세요.

### strict mode

Android에서 StrictMode를 사용하면 여러가지 문제를 감지 할 수 있습니다.

- 제거 가능한 객체가 제거되지 않을 때
- 메인 쓰레드에서 수행되는 파일 읽기나 네트워크 요청
- URI 노출
- ...

이러한 문제가 발견 될 때마다 구성에 따라서 적잘한 로그를 출력하거나 어플리케이션을 중단 할 수 있습니다.

디버그 빌드에서만 사용하도록 설정하고 `detectAll()` 메소드를 사용하여 모든 종류의 문제를 감지하는 것이 좋습니다.

```java
if (BuildConfig.DEBUG) {
    StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
            .detectAll()
            .penaltyLog()
            .build());
    StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
            .detectAll()
            .penaltyLog()
            .build());
}
```

다음은 SQLiteCursor를 닫지 않은 로그의 예입니다.

```
StrictMode:
A resource was acquired at attached stack trace but never released.
See java.io.Closeable for information on avoiding resource leaks.
java.lang.Throwable: Explicit termination method 'close' not called
        at dalvik.system.CloseGuard.open(CloseGuard.java:184)
        at android.database.CursorWindow.<init>(CursorWindow.java:111)
        at android.database.AbstractWindowedCursor.clearOrCreateWindow(AbstractWindowedCursor.java:198)
        at android.database.sqlite.SQLiteCursor.fillWindow(SQLiteCursor.java:139)
        at android.database.sqlite.SQLiteCursor.getCount(SQLiteCursor.java:133)
        at android.database.AbstractCursor.moveToPosition(AbstractCursor.java:197)
        at android.database.AbstractCursor.moveToFirst(AbstractCursor.java:237)
        at com.dd.template.MainActivity.onCreate(MainActivity.java:124)
```

> 템플릿 프로젝트에서 [StrictMode](https://github.com/dmytrodanylyk/template/blob/master/app/src/main/java/com/dd/template/TemplateApplication.java#L12)를 확인해보세요.
