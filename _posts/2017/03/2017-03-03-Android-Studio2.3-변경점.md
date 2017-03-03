---
layout: post
category : Android
title : "[Android Studio] Android Studio 2.3 변경점"
description : ""
tags : [Android, Android Studio]
---

{% include JB/setup %}

# [Android Studio 2.3 변경점](http://android-developers.googleblog.com/2017/03/android-studio-2-3.html)

## Build

### Instant Run 개선 및 UI 변경

- Instant Run을 일부 변경하여 기능의 안정성을 향상
- Instant Run의 구현이 안정성 향상을 위해 크게 변경되었고 앱의 시동 지연도 제거되었음
- [자세히 알아보기](https://developer.android.com/studio/run/index.html#instant-run)

![](https://1.bp.blogspot.com/-tiW6x503J_c/WLhSflDxIHI/AAAAAAAAD8A/EUwQJ8GmeCQzTT9OBG-KR8x9qumvVyhcwCLcB/s640/Screen%2BShot%2B2017-03-02%2Bat%2B9.12.10%2BAM.png)

### Build Cache

- AAR 및 미리 처리된 외부 라이브러리를 캐싱하여 더 빠른 clean build로 이어짐
- Android Studio 2.3에서 기본적으로 사용하도록 설정
- [자세히 알아보기](https://developer.android.com/studio/build/build-cache.html)

## Design

### Constraint Layout에 체인 및 비율 지원

- Android Studio 2.3에는 ConstraintLayout의 안정적인 릴리즈 버전이 포함되어있음
- ConstraintLayout 안에서 여러 개의 View를 양 방향으로 연결 할 수 있음
- 이 방법은 여러 개의 View를 일정 거리로 배치하려는 경우에 유용함
- [자세히 알아보기](https://developer.android.com/training/constraint-layout/index.html#constrain-chain)

![](https://2.bp.blogspot.com/-PbL5QK-UjWE/WLhStg49nTI/AAAAAAAAD8I/N08ZsVdcqeEQN4sVRj4p2k5KTPgqVRt6QCLcB/s640/chains_support.gif)

- ConstraintLayout은 비율을 지원하기 때문에 포함된 레이아웃이 확장 및 축소 될 때 위젯의 종회비를 유지하려는 경우에 유용함
- [비율에 대해 자세히 알아보기](https://developer.android.com/training/constraint-layout/index.html#ratio)
- ConstraintLayout의 체인과 비율 모두 ConstraintSet API를 사용한 프로그래밍 방식의 작성을 지원할 수 있음

### Layout Editor Palette

- Layout Editor에 업데이트 된 위젯 Palette를 사용하면 레이아웃 위젯을 찾기 위해 검색, 정렬 및 필터링을 수행 할 수 있으며, 디자인 화면으로 드래그하기 전에 미리 위젯을 볼 수 있음
- [자세히 알아보기](https://developer.android.com/studio/write/layout-editor.html)
![](https://2.bp.blogspot.com/-jSZ8PNpvBiA/WLhTRsb9WeI/AAAAAAAAD8M/Y51t1L6PeNYkYzlIljJglYNjIwtM6a6UwCLcB/s640/Screen%2BShot%2B2017-03-02%2Bat%2B9.14.58%2BAM.png)

### Layout 즐겨찾기

- 레이아웃 편집기 속성 패널에서 즐겨쓰는 속성을 저장
- 고급 패널에서 속성에 별표를 표시하면 즐겨찾기 섹션 아래에 나타남
- [자세히 알아보기](https://developer.android.com/studio/write/layout-editor.html#edit-properties)
![](https://4.bp.blogspot.com/-7yoX1jWDGlE/WLhTni8yKOI/AAAAAAAAD8Q/h6o5zBW3KLAvJG_o6gGySoYukzofXw_TACLcB/s640/fav_attributes.gif)

### WebP 지원

- Android Studio는 APK의 용량을 절약 할 수 있도록 프로젝트의 PNG 이미지에서 WebP 이미지를 생성
- WebP 무손실 포멧은 PNG보다 최대 25%작으며 Android Studio의 변환 마법사를 사용하여 손실 된 WebP 인코딩도 검사 할 수 있음
- PNG 파일을 마우스 오른쪽 버튼으로 클릭하여 WebP로 변환 가능
- 이미지를 편집해야하는 경우 프로젝트의 모든 WebP 파일을 마우스 오른쪽 버튼으로 클릭하여 PNG로 다시 변환 할 수 있음
- [자세히 알아보기](https://developer.android.com/studio/write/convert-webp.html)
![](https://4.bp.blogspot.com/-3K11mY1HWU0/WLhT0txvfSI/AAAAAAAAD8U/3958U0j0-Uomhux5IHPd4_a63aBtTx_NwCLcB/s640/Screen%2BShot%2B2017-03-02%2Bat%2B9.17.47%2BAM.png)

### Material 아이콘 마법사 업데이트

- 업데이트 된 vector asset 마법사는 검색 및 필터링을 지원하고 각 아이콘 asset에 대한 라벨을 포함
- [자세히 알아보기](https://developer.android.com/studio/write/vector-asset-studio.html#materialicon)
![](https://1.bp.blogspot.com/-HHsLdicp6u4/WLhT-H1An0I/AAAAAAAAD8Y/YcPIiaLzCs4AJImz5SYqeenzribDXrWowCLcB/s640/Screen%2BShot%2B2017-03-02%2Bat%2B9.18.30%2BAM.png)

## Develop

### Lint Baseline

- Android Studio 2.3에서 lint 경고를 프로젝트 단위로 설정 할 수 있음
- 그 시점부터 lint는 새로운 문제만 보고
- 앱이 많은 lint 문제를 가지고 있지만, 새로운 lint 문제를 해결하는 데만 집중하려는 경우 유용
- 이번 릴리즈에서 추가 된 [새로운 Lint 검사 및 어노테이션](https://developer.android.com/studio/releases/index.html)에 대해 [자세히 알아보기](https://developer.android.com/studio/write/lint.html#snapshot)

![](https://1.bp.blogspot.com/-jlpxxHZgdsw/WLhUHgKLJuI/AAAAAAAAD8c/TLx1o3DUuFIoLWIWLzHEqgtXbLLNmEp5gCLcB/s640/Screen%2BShot%2B2017-03-02%2Bat%2B9.19.09%2BAM.png)

### App Links Assistant

- App Links Assistant를 사용하면 URL에 대한 인텐트 필터를 쉽게 만들고 Digital Asset Links 파일을 통해 앱에 대한 웹사이트 연결을 선언 할 수 있으며 Android 앱 링크 지원을 테스트 할 수 있음
- App Link Assistant에 액세스하려면 Tools → App Link Assistant
- [자세히 알아보기](https://developer.android.com/studio/write/app-link-indexing.html)
![](https://4.bp.blogspot.com/-9msX55F8JgU/WLhUXi8TPAI/AAAAAAAAD8g/Z067RdnFg3MzIPq-rEppNFVF7fAs80-0wCLcB/s640/Screen%2BShot%2B2017-03-02%2Bat%2B9.19.52%2BAM.png)

### Template 업데이트 

- 기본적으로 RelativeLayout을 포함하는 Android Studio 2.3의 모든 템플릿은 이제 ConstraintLayout을 사용
- [Templates](https://developer.android.com/studio/projects/templates.html)과 [ConstraintLayout](https://developer.android.com/training/constraint-layout/index.html)
![](https://1.bp.blogspot.com/-J-zTDlFeDuo/WLhUkDxBcCI/AAAAAAAAD8k/0y0o1zNrBdEOJr8gFaNTxjQQniFdXaA4wCLcB/s640/Screen%2BShot%2B2017-03-02%2Bat%2B9.20.57%2BAM.png)

### IntelliJ 플랫폼 업데이트
- Android Studio 2.3에는 업데이트 된 inspection window 및 알림 시스템과 같은 향상된 기능이 포함 된 IntelliJ 2016.2 릴리스가 포함되어 있음

## Test

### Android Emulator 복사 및 붙여 넣기

- 복사 및 붙여넣기 기능을 최신 에뮬레이터 (v25.3.1)에 추가
- 안드로이드 에뮬레이터와 호스트 운영체제 사이에 공유 클립 보드가있어 두 환경 사이에서 텍스트를 복사 할 수 있음
- 복사 및 붙여넣기는 x86 Google API 에뮬레이터 시스템 이미지 API 레벨 19 (Android 4.4 - Kitkat) 이상에서 작동

![](https://1.bp.blogspot.com/-45Tto0NWxQg/WLhVMkO969I/AAAAAAAAD8o/6HT1lLNW2Xsy1d7ilh-kEStBYZPfaqYKACLcB/s640/emulator_copy_paste.gif)

### Android Emulator Command Line Tool

- Android SDK Tools 25.3부터는 SDK 도구 폴더의 에뮬레이터를 별도의 에뮬레이터 디렉토리로 옮기고 "android avd" 명령을 독립형 avdmanager 명령으로 대체
- 에뮬레이터 및 "android avd"에 대한 이전 Command Line 매개 변수는 업데이트 된 도구에서 작동
- 명령 줄을 통해 직접 Android 가상 장치(AVD)를 만드는 경우 해당 스크립트를 업데이트 해야함
- Android Studio 2.3을 통해 Android Emulator를 사용하는 경우 이러한 변경 사항이 워크 플로에 영향을 미치지 않음
- [자세히 알아보기](https://developer.android.com/studio/releases/sdk-tools.html)


