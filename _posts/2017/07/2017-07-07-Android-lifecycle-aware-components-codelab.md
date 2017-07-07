---
layout: post
category : Android
title : "[Architecture] Android lifecycle-aware components codelab"
description : ""
tags : [Android, Architecture, lifecycle]
---

{% include JB/setup %}

> *이 문서는 [Android lifecycle-aware components codelab
](https://codelabs.developers.google.com/codelabs/android-lifecycles/index.html?index=..%2F..%2Findex#0)를 번역한 문서입니다.*

# Android lifecycle-aware components codelab

## 1. 소개

### 구성 요소

아키텍처 구성요소는 견고하고, 테스트 가능하며 유지보수가 가능한 방식으로 앱을 구성하는데 도움이 되는 Android 라이브러리입니다.

> 주의: 아키텍처 구성요소 라이브러리는 개발이 알파버전입니다. API는 일반적으로 사용하기 전에 변경 될 수 있으며 안정성 또는 성능 문제가 발생할 수 있습니다.

이 코드랩은 Android 앱 개발을 위한 다음의 lifecycle-aware 아키텍처 구성요소를 소개합니다.

- ViewModel : 특정 라이프사이클에 바인딩 된 객체를 생성하고 검색하는 방법을 제공합니다. ViewModel은 뷰의 데이터 상태를 저장하고 데이터 리포지토리나 비지니스 로직을 처리하는 도메인 계층과 같은 별도의 구성요소와 통신합니다. 이 항복의 입문서를 읽으려면 ViewModel을 참조하세요.
- LifecycleOwner/LifecycleRegistryOwner : LifecycleOwner 및 LifecycleRegistryOwner는 LifecycleActivity 및 LifecycleFragment 클래스로 구현되는 인터페이스입니다. 이러한 인터페이스를 구현하는 소유자 객체에 다른 구성요소를 구독시켜 소유자 라이프사이클의 변경 사항을 관찰할 수 있습니다. 이 항목의 소개 가이드를 읽으려면 Lifecycle 처리를 참조하세요.
- LiveData : 명시적이고 엄격한 의존성 경로를 만들지 않고 앱의 여러 구성요소에서 데이터 변경사항을 관찰할 수 있도록합니다. LiveData는 Activity, Fragment, Service 또는 앱에 정의된 LifecycleOwner을 비롯하여 앱 구성요소의 복잡한 Lifecycle을 존중합니다. LiveData는 중지된 LifecycleOwner 객체에 대한 구독을 일시 중지하고 중지 완료된 LifecycleOwner 객체에 대한 구독을 취소하는 등의 관찰자 구독을 관리합니다. 이 항목의 소개 가이드는 LiveData를 참조하세요.

### 우리는 이것을 만듭니다

이 코드랩에서는 위에서 설명한 각 구성요소의 예를 구현합니다. 샘플 앱으로 시작하여 일련의 단계를 통해 코드를 추가하고 진행하면서 다양한 아키텍처 구성요소를 통합니다.

### 필요사항

Android Studio 2.3 이상
Android Activity Lifecycle에 대해 잘 알고 있어야합니다.

## 2. 1단계 - 환경 설정

이 단계에서는 전체 코드랩에 대한 소스를 다운로드 한 다음 간단한 예제 어플리케이션을 실행합니다.

이 코드랩에 대한 모든 코드를 다운로드하려면 다음 버튼을 클릭하세요.

[Download source code](https://github.com/googlecodelabs/android-lifecycles)

1.코드의 압축을 푼 다음 Android Studio 버전 2.3 이상에서 오픈합니다.

2. 장치 또는 에뮬레이터에서 **Step 1** 실행 구성을 실행하세요.

![](https://codelabs.developers.google.com/codelabs/android-lifecycles/img/b0c91bf76f7b9a13.png)

앱이 실행되고 다음 스크린 샷과 유사한 화면이 표시됩니다.

![](https://codelabs.developers.google.com/codelabs/android-lifecycles/img/d928fd8587074072.png)

3. 화면을 회전 시키면 타이머가 재설정됩니다!

화면이 회전을 하여도 데이터가 유지되도록 앱을 업데이트해야합니다. 이 클래스의 인스턴스는 화면 회전과 같은 구성 변경사항을 유지하기 때문에 ViewModel을 사용해야합니다.

## 3. 2단계 - ViewModel 추가

이 단계에서는 ViewModel을 이용하여 화면 회전에서 상태를 유지하고 이전 단계에서 관찰한 행동을 처리합니다. 이전 단계에서 타이머를 표시하는 Activity를 실행했습니다. 이 타이머는 화면 회전과 같은 구성 변경이나 Activity가 파괴 될 때 타이머가 리셋됩니다.

ViewModel을 사용하면 Activity 또는 Fragment의 전체 Lifecycle에 걸쳐 데이터를 유지할 수 있습니다. 이전 단계에서 설명한 것처럼 Activity는 앱 데이터를 관리하기에 좋은 선택이 아닙니다. Activity 및 Fragment는 사용자가 앱과 상호 작용을 할 때 생성되고 파괴되는 수명이 짧은 객체입니다. ViewModel은 네트워크 통신 뿐만 아니라 데이터 조작 및 지속성과 관련된 작업을 관리하는데 더 적합합니다.

### ViewModel을 사용하여 Chronometer 상태 유지

`ChronoActivity2.java`를 열고 클래스가 ViewModel을 검색하여 사용하는 방법을 확인하세요.

```java
ChronometerViewModel chronometerViewModel
        = ViewModelProviders.of(this).get(ChronometerViewModel.class);
```

`this`는 LifecycleOwner의 인스턴스를 나타냅니다. 프레임워크는 LifecycleOwner의 범위가 살아있는 한 ViewModel을 활성 상태를 유지합니다. ViewModel은 화면 회전과 같은 구성변경으로 소유자가 삭제 된 경우에도 삭제되지 않습니다. 다음 그림과 같이 소유자의 새 인스턴스가 기존 ViewModel에 다시 연결됩니다.

![](https://codelabs.developers.google.com/codelabs/android-lifecycles/img/8efa40941430fc4c.png)

> 주의: Activity 또는 Fragment의 범위는 생성된 상태에서 완료된 상태(또는 종료된 상태)로 진행되며, 이를 파괴된 상태로 현동해서는 안됩니다. 디바이스가 회전 될 때, Activity는 파괴되지만 연관된 ViewModel 인스턴스는 그렇지 않습니다.

### 다음을 시도해보세요

다음 작업 중 하나를 수행 할 때 앱을 실행하고 타이머가 재 설정되지 않는지 확인하세요.

1. 화면을 회전하세요.
2. 다른 앱으로 이동한 다음 돌아가보세요.

![](https://codelabs.developers.google.com/codelabs/android-lifecycles/img/5fcb2c457ab6ae30.png)

그러나 사용자나 시스템이 앱을 종료하면 타이머가 재 설정됩니다.

> 주의: Fragment또는 Activity와 같은 Lifecycle 소유자의 수명동안 ViewModel의 인스턴스가 메모리에 지속됩니다. 시스템엔 ViewModel의 인스턴스를 장기간 저장하지 않습니다.

## 4. 3단계 - LiveData를 사용하여 데이터 줄 바꾸기

이 단계에서는 이전 단계에서 사용한 Chronometer를 타이머로 사용하는 사용자 정의 카메라로 바꾸고 매 초마다 UI를 업데이트합니다. Timer는 향후에 작업을 반복하여 예약하는데 사용할 수 있는 java.util 클래스입니다. 이 로직을 LiveDataTimerViewModel 클래수에 추가하고 사용자와 UI 간의 상호작용을 관리하는 데 집중 할 수 있는 Activity를 추가합니다.

타이머가 알람을 보내면 Acitivty가 UI를 업데이트합니다. 메모리 누수를 피하기 위해 ViewModel에는 Activity에 대한 참조가 포함되지 않습니다. 예를 들어, 화면 회전과 같은 구성 변경으로 인해 GC에 의해 처리되어야하는 Activity를 ViewModel이 참조하고 있을 수 있습니다. 시스템은 해당 Activity 또는 Lifecycle 소유자가 더 이상 존재하지 않을 때까지 ViewModel의 인스턴스를 보유합니다.

> 주의: ViewModel에서 Context 또는 View에 대한 참조를 저장하면 메모리 누수가 발생 할 수 있습니다. Context 또는 View 클래스의 인스턴스를 참조하는 필드를 사용하지 마세요. onCleared() 메소드는 수명주기가 긴 다른 객체에 대한 참조를 등록 취소하거나 지우는데 유용하지만, Context 또는 View 객체에 대한 참조를 지우는데 유용하지 않습니다.

ViewModel에서 직접 View를 수정하는 대신 Activity 또는 Fragment를 구성하여 데이터 소스를 관찰하고 변경된 데이터를 수신합니다. 이러한 패턴을 관찰자(observer) 패턴이라 합니다.

> Note: 데이터를 관찰 가능 데이터로 표시하려면 데이터를 LiveData 클래스로 래핑합니다.

데이터 바인딩 라이브러리나 RxJava와 같은 반응형 라이브러리를 사용했다면 관찰자 패턴을 잘 알고 있을것입니다. LiveData는 Lifecycle을 인식하는 특수한 관찰 가능한 클래스이며 활성된 관찰자에게만 알립니다.

### LifecycleOwner

`ChronoActivity3.java`는 Lifecycle 상태를 제공하는 LifecycleActivity의 인스턴스입니다. 다음은 클래스 선언입니다.

```java
public class LifecycleActivity extends FragmentActivity implements LifecycleRegistryOwner {...}
```

LifecycleRegistryOwner는 ViewModel 및 LiveData 인스턴스의 Lifecycle를 Activity 또는 Fragment에 바인딩하는데 사용됩니다. Fragment에 해당하는 클래스는 LifecycleFragment입니다.

### ChronoActivity 업데이트

1. `ChronoActivity3` 클래스에 subscribe() 메소드에 구독을 만드는 다음 코드를 추가합니다.

```java
mLiveDataTimerViewModel.getElapsedTime().observe(this, elapsedTimeObserver);
```

2. 그런 다음 `LiveDataTimerViewModel` 클래스에서 새 경과 시간 값을 설정합니다. 다음 주석을 찾으세요.

```java
//TODO set the new value
```

주석을 다음 소스로 대체합니다.

```java
mElapsedTime.setValue(newValue);
```

3. 앱을 실행하고 Android Studio에서 Android Monitor를 엽니다. 다른 앱으로 이동하지 않는 한 로그가 1초마다 업데이트 됩니다. 장치가 다중 창 모드를 지원하는 경우, 그것을 사용하려고 할 수 있습니다. 화면을 회전해도 앱 작동 방식에는 영향을 미치지 않습니다.

![](https://codelabs.developers.google.com/codelabs/android-lifecycles/img/9913571bad30fed9.png)

> Note: LiveData 객체는 Activity 또는 LifecycleOwner이 활성 상태일 때만 업데이트를 보냅니다. 다른 앱으로 이동하면 돌아올 때까지 로그 메시지가 일시중지됩니다. LiveData 객체는 해당 Lifecycle 소유자가 STARTED 또는 RESUMED 일 때만 구독을 활성으로 간주합니다.

## 5. 4단계 - Lifecycle 이벤트 구독

많은 Android 구성요소 및 라이브러리에서 다음을 수행해야합니다.

1. 구성요소 또는 라이브러리를 구독하고나 초기화
2. 구성요소 또는 라이브러리 구독을 취소하거나 중지

위의 단계를 완료하지 못하면 메모리 누수 및 미묘한 버그가 발생 할 수 있습니다.

Lifecycle 소유자 객체는 lifecycle-aware 구성요소의 새 인스턴스로 전달되여 Lifecycle의 현재 상태를 알 수 있도록 합니다.

다음 소스를 사용하여 Lifecycle의 현재 상태를 질의 할 수 있습니다.

```java
lifecycleOwner.getLifecycle().getCurrentState()
```

위의 소스는 Lifecycle.State.RESUME 또는 Lifecycle.State.DESTROYED와 같은 상태를 반환합니다.

LifecycleObserver를 구현하는 lifecycle-aware 객체는 Lifecycle 소유자의 상태 변화를 관찰 할 수 있습니다.

```java
lifecycleOwner.getLifecycle().addObserver(this);
```

필요한 경우 Lifecycle 상태에 따라 적절한 메소드를 호출하도록 어노테이션을 달 수 있습니다.

```java
@OnLifecycleEvent(Lifecycle.EVENT.ON_RESUME)
void addLocationListener() { ... }
```

### lifecycle-aware 구성요소 생성

이 단계에서는 Activity Lifecycle 소유자에게 반응하는 구성요소를 만듭니다. Fragment를 Lifecycle 소유자로 사용 할 때도 이와 유사한 원칙과 단계가 적용됩니다.

 - Android 프레임워크의 LocationManager를 사용하여 현재 위도와 경도를 가져와서 사용자에게 표시합니다. 이 추가 기능을 통해 다음을 수행 할 수 있습니다.
 - 변경사항을 구독하고 LiveData를 사용하여 UI를 자동으로 업데이트합니다. Activity 상태의 변경에 따라 등록 및 등륵을 취소하는 LocationManager의 래퍼를 만듭니다.

일반적으로 Activity의 onStart() 또는 onResume() 메소드의 변경 사항에 LocationManager를 등록하고 onStop() 또는 onPause() 메소드에서 리스너를 제거합니다.

```java
// Typical use, within an activity.

@Override
protected void onResume() {
    mLocationManager.requestLocationUpdates(LocationManager.GPS_PROVIDER, 0, 0, mListener);
}

@Override
protected void onPause() {
    mLocationManager.removeUpdates(mListener);
}
```

BoundLocationMnager라는 클래스에서 LifecycleRegistryOwner라는 LifecycleOwner 구현을 사용합니다. BoundLocationManager 클래스의 이름은 클래스의 인스턴스가 Activity Lifecycle에 바인딩 된다는 사실을 참조합니다.

클래스가 Acitivty의 Lifecycle을 관찰하려면 관찰자로 해당 객체를 추가해야합니다. 이렇게 하려면 생성자에 다음 코드를 추가하여 BoundLocationManger 객체에 Lifecycle을 관찰하도록 지시합니다.

```java
lifecycleOwner.getLifecycle().addObserver(this);
```

Lifecycle이 변경 될 때 메소드를호출하려면 @OnLifecycleEvent 어노테이션을 사용 할 수 있습니다. BoundLocationListener 클래스의 다음 어노테이션을 사용하여 addLocationListener() 및 removeLocationListener() 메소드를 업데이트합니다.

```java
@OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
void addLocationListener() {
    ...
}
@OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
void removeLocationListener() {
    ...
}
```

> Note: observer는 provider의 현재 상태가 되어, 생성자로부터 addLocationListener()을 호출 할 필요는 없습니다. 관찰자가 Lifecycle 소유자에게 추가되면 당신을 위해 호출합니다.

앱을 실행하고 기기를 회전 할 때 로그 모니터에 다음 액션이 표시되는지 확인합니다.

```
D/BoundLocationMgr: Listener added
D/BoundLocationMgr: Listener removed
D/BoundLocationMgr: Listener added
D/BoundLocationMgr: Listener removed
```

Android 에뮬레이터를 사용하여 기기의 위치 변경을 시뮬레이트 합니다.(확장된 컨트롤을 표시하려면 세 개의 점을 클릭하세요.) 텍스트 뷰가 변경되면서 업데이트 됩니다.

![](https://codelabs.developers.google.com/codelabs/android-lifecycles/img/62bc0f2aa883739c.png)


## 6. 5단계 - Fragment 사이에서 ViewModel 공유

### Fragment 간의 ViewModel 공유

Fragment과 ViewModel을 사용하여 통신을 가능하게 하기 위해 다음의 추가 단계를 완료하세요.

1. 단일 Activity
2. 2개의 Fragment 인스턴스, 각각 SeekBar가 존재
3. LiveData 필드가 있는 단일 ViewModel

이 단계를 실행하고 서로 독립적인 SeekBar를 확인합니다.

![](https://codelabs.developers.google.com/codelabs/android-lifecycles/img/3779b3df20c610d4.png)

하나의 SeekBar가 변경되면 다른 SeekBar가 업데이트 되도록 ViewModel을 사용하여 Fragment를 연결합니다.

![](https://codelabs.developers.google.com/codelabs/android-lifecycles/img/a6c30f446ff0869a.png)




