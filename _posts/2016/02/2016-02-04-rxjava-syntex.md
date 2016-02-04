---
layout: post
category : Android
title : "[RxJava] RxJava Document 정리"
description : ""
tags : [Android, RxJava, ReactiveX]
---

{% include JB/setup %}

하단의 Reference를 참조하여 작성, 주요 사용되는 RxJava 클래스와 Observable에서 호출되는 메소드를 다룸

## RxJava Class

- `Observable`  
이벤트를 발생시키는 주체, onNext / onCompleted / onError를 이용하여 이벤트를 발생 시킴

- `Subscriber`  
이벤트를 전달받는 객체

- `PublishSubject`  
구독한 시점으로 부터(subscribe 호출) 발생되는 이벤트(onNext, onError, onCompleted)를 전달 받음

- `BehaviorSubject`  
구독 전 (subscribe 호출 전) 발생된 이벤트가 한 건이라도 있으면 구독 시점에 해당 이벤트(한 건만)를 전달 받음

- `CompositeSubscription`  
Subscriber를 그룹화 함, add로 Subscriber를 추가하며 clear로 unsubscribe 처리

## Observable Method

- `just(T...)`
인자로 전달 받은 객체를 그냥 단순하게 subscribe onNext에 전달

- `from(Iterable<? extends T>)`  
List에 저장된 Item을 하나씩 전달

- `flatMap(Func1<? super T,? extends Observable<? extends R>>)`  
T를 매개변수로 전달받아 R로 리턴  
인자로 전달 받은 T를 다시 Observable로 포장하여 리턴 처리  
관찰하고 있는 이벤트를 변형  

- `concanMap(Func1<? super T,? extends Observable<? extends R>>)`
flatMap과 유사, flatMap은 결과 처리의 순서를 신경쓰지 않는 반면에 concanMap는 이벤트의 처리 결과가 순서대로 출력 (참조)

- `filter(Func1<? super T, Boolean>)`  
return이 true 일 때 이벤트 발생

- `zip(Observable<? extends T1>, Observable<? extends T2>, Func2<? super T1,? super T2,? extends R>)`  
Observable 마지막 인자인 Func2로 값을 조합하여 넘김  
T1과 T2를 조합하여 새로운 R을 생성  

- `buffer(int)`  
지정된 시간동안 발생한 이벤트를 List로 저장하고 있다가 이벤트를 발생 시킴

- `buffer(Observable<B> boundary)`  
발생한 데이터를 저장하고 있다가 boundary에 지정된 시간이 지나면 이벤트를 발생 시킴

- `debounce(long, TimeUnit)`  
이벤트 발생 종료 후 지정된 시간동안 이벤트가 발생하지 않으면 이벤트를 발생 시킴

- `asObservable()`  
Observable 객체를 전달 받음

- `skip(int)`  
int로 지정된 개수 만큼의 이벤트를 무시하고 그 이후의 이벤트만 발생시킴

- `combineLatest(Observable<? extends T1>, Observable<? extends T2>, Func2<? super T1, ? super T2, ? extends R>)`  
모든 Observable에서 이벤트가 발생되면 Func2로 전달하여 이벤트를 처리하고 새로운 이벤트를 발생 시킴

- `merge(Observable<? extends Observable<? extends T>>)`  
다수의 Observable를 병합하여 이벤트가 발생 된 순서대로 이벤트를 전달, 한 Observable에서 error가 발생하면 다른 Observable에서 발생된 이벤트는 전달하지 않음

- `timer(long, TimeUnit)`  
지정된 시간만큼 이벤트 전달을 딜레이 시킴

- `interval(long, TimeUnit)`  
이벤트를 반복하여 발생 시키지만 지정된 시간동안 딜레이 시킴

- `interval(long, long, TimeUnit)`  
최초 첫번째 인자 값 만큼 시간을 딜레이시키고 그 이후에는 2번째 인자에 지정된 시간 만큼 딜레이 시킴

- `take(int)`  
인자로 지정된 개수 만큼 이벤트를 발생 시키고 그 이후 이벤트는 발생시키지 않음(onCompleted 발생)  
interval과 같이 사용하면 지정된 시간만큼 해당 이벤트를 반복하여 발생시키지만 take로 지정된 개수만 발생

- `retryWhen(Func1<? super Observable<? extends Throwable>,? extends Observable<?>>)`  
onError 이벤트가 발생하게되면 지정된 시간만큼 딜레이 시킨 뒤 재시도

- `defer(Func0<Observable<T>>)`  
subscribe가 호출 되는 순간 인자로 지정된 Func이 실행되고 리턴 받은 Observable의 이벤트를 전달

- `doOnTerminate(Action0)` 
onCompleted나 onError가 발생하면 호출되는 메소드, 인자 값으로 처리 할 작업을 넘김

- `doOnUnsubscribe(Action0)`  
unsubscribed가 호출되면 호출되는 메소드, onCompleted나 onError가 발생하면 unsubscribed가 발생하게 되는데 `doOnTerminate(Action0)`와 비슷하지만 unsubscribed를 강제적으로 호출해야할 경우가 발생하기 때문에 해당 메소드 사용을 추천

## Reference

[RxJava with Android - 1 - RxJava 사용해보기](http://gaemi.github.io/android/2015/05/20/RxJava%20with%20Android%20-%201%20-%20RxJava%20%EC%82%AC%EC%9A%A9%ED%95%B4%EB%B3%B4%EA%B8%B0.html)

[RxJava-Android-Samples](https://github.com/kaushikgopal/RxJava-Android-Samples)

[ReactiveX Docs](http://reactivex.io/documentation/observable.html)
