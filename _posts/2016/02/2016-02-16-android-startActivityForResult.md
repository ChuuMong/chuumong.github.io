---
layout: post
category : Android
title : "[Activity] startActivityForResult(), onActivityResult()"
description : ""
tags : [Android, Activity]
---

{% include JB/setup %}

## `startActivityForResult()`

- Activity A가 `startActivityForResult()`를 통해서 Activity B를 호출하면, Activity B 생성
- 이때 Activity B에서는 작업을 진행하다가 `setResult(Int, Intent)`를 이용하여 Extra 값을 Activity A에 넘김
- 그러면 Activity A에서는 `onActivityResult()`를 통해서 Extra 값을 받을 수 있음

### Activity A 구현 사항

1. int값의 requestCode 값 설정
2. Intent를 만들어 Activity B를 실행  
 - `startActivityForResult()` 이용
3. onActivityResult()를 통해 각 requestCode값에 해당하는 Extra 값을 처리

### Activity B 구현 사항

1. Intent 생성
2. Bundle 객체를 생성하여 값을 저장하고 Intent의 `putExtras(Bundle)`를 호출하여 Bundle 객체를 저장
3. `setResult(Int, Intent)` 호출하여 Activity A로 Intent 전달
