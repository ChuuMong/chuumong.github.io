---
layout: post
category : Android
title : "[Android] 어플리케이션 버전 관리"
description : ""
tags : [Android, Version]
---

{% include JB/setup %}

iOS 공부 중 [여기](http://www.raywenderlich.com/97014)에 버전 관리에 대한 글이 있어 옮김


## Intro
일반적으로 Android에서 버전 관리는 2가지로 이루어지게 된다.

```gradle
versionCode 1
versionName "1.0"
```

## versionCode
`versionCode`는 다른 버전과 상대적인 값을 나타내는 정수형 값이다. 정수형으로 선언되기 때문에 프로그램상에서 현재의 버전이 더 높은 버전인지 또는 낮은 버전인지 확인이 가능하기 때문에 어플리케이션에 업데이트가 발생하면 해당 값을 증가시켜주면 된다.

메이저 업데이트인지 마이너 업데이트인지 상관없이 업데이트 시 증가해주면 되며 사용자에게 보여지는 값은 아니고 시스템적으로 버전의 업데이트 여부를 확인하는 용으로 사용된다.

## versionName

기본적으로 안드로이드 프로젝트를 생서하게 되면 `versionName`은 "1.0"으로 잡히게 된다. 이 값은 `String`으로 지정되며 사용자에게 보여주기 위한 값이다.

보통 아래 그림과 같이 **major.minor.patch**으로 이루어 지게 된다.

![버전](http://cdn5.raywenderlich.com/wp-content/uploads/2015/03/sem_versioning.png)


### major

major는 기존의 기능이 수정되어 하위 버전과 호완이 되어지지 않을 때 값을 증가 시킨다. 이때 minor와 patch는 값이 0이 되어지게 된다.

### minor

minor는 기존의 기능이 변경되지 않고 새로운 기능이 추가되었을 때 값을 증가시킨다. 이때 patch는 값이 0이 되어지게 된다.

### patch

patch는 버그가 수정되는 경우, 값이 증가한다.


## Reference

[How to Use CocoaPods with Swift](http://www.raywenderlich.com/97014)  
[app 버전 관리](http://linuxism.tistory.com/920)




