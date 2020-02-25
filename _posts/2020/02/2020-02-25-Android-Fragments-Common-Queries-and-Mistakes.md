이 글은 [Android Fragments Common Queries And Mistakes
](https://levelup.gitconnected.com/android-fragments-common-queries-and-mistakes-915b4a1f34ce)
의 번역본이며 문제가 발생 할 시에 제거 될 수 있습니다.

안드로이드에서 `Fragment` class는 다이나믹한 UI를 작성하는데 사용되며 Activity 내에서 사용해야 합니다. Fragment를 사용하는 가장 큰 장점은 여러 화면 크기의 UI를 만드는 작업을 단순화 한다는 것입니다. Activity에는 여러 개의 Fragment가 포함 될 수 있습니다.

이 설명은 Fragment를 쉽고 좋게 만듭니다. 그렇게 빠르지는 않지만 더 많은 관련이 있습니다. 이 기사에서는 Fragment르 사용하는 동안 주요 필요사항과 일반적인 실수를 다룰 것입니다.

*참고 : Fragment 및 Fragment의 LifeCycle callback에 대한 기본 지식이 있다고 가정합니다. 또한 두 Fragment 사이의 통시을 구현하는 방법을 알고 있다고 가정합니다. 이 글은 그 이상입니다.*

# 장애물

다음은 이미 직면했거나 향후에 발생 할 수 있는 Fragment와 관련된 몇 가지 장애물입니다.

- FragmentManager : `getSupportFragmentManager()`와 `getChildFragmentManager()`. 메모리 누출을 방지하고 사용 시 사용할 항목
- DialogFragment, ChildFragment, BottomSheetFragment에서 부모 Fragment으로의 콜백
- ViewPager에서 Fragment를 사용 할 때 FragmentStateAdapter와 FragmentPagerAdapter 중 어떤 것을 사용할 지
- 언제 FragmentTransaction의 `add()`와 `replace()`를 사용 할 지
- Fragment receiver, broadcast 및 메모리 누수
- Fragment BottomBarNavigation와 drawer를 처리하는 방법
- `commit()`와 `commitAllowingStateLoss()`
- Fragment 옵션 메뉴
- Fragment `getActivity()`, `getView()` 및 NullPointers Exceptions
- 중첩된 Fragment가 있는 `onActivityResult()`
- Fragment 및 Bundle
- Back Navigation

## getSupportFragmentManager 및 getChildFragmentManager

FragmentManager는 Fragment를 추가, 제거 또는 교체하기 위한 트랜잭션을 만드는데 사용되는 프레임워크에 의해 제공되는 클래스입니다.

- `getSupportFragmentManager()`는 Activity와 연관이 있습니다. Activity를 위한 FragmentManager라 생각하세요.

따라서, Activity에서 ViewPager, BottomSheetFragment 및 DialogFragment를 사용 할 때 `getSupportFragmentManager()`를 사용합니다.

예: 
```
BottomDialogFragment bottomSheetDialog = BottomDialogFragment.getInstance();
bottomSheetDialog.show(getSupportFragmentManager(), "Custom Bottom Sheet");
```

- `getChildFragmentManager()`는 Fragment와 연관이 있습니다.

Fragment 안에 ViewPager가 잇으면 `getChildFragmentManager()`를 사용해야 합니다.

예:
```
FragmentManager cfManager=getChildFragmentManager();
viewPagerAdapter = new ViewPagerAdapter(cfManager);
```

추가적인 이해를 위한 [공식 링크](https://developer.android.com/reference/android/support/v4/app/FragmentManager.html)는 다음과 같습니다.

Fragment 내에서 ViewPager를 사용 할 때 사람들이 흔히 저지르는 실수는 Activity의 Fragment 관리자인 `getSupportFragmentManager()`를 전달하기 때문에 메모리 누수 또는 ViewPager가 제대로 업데이트 되지 않는 등의 문제가 발생합니다.

Fragment에서 `getSupportFragmentManager()`를 사용하여 발생하는 가장 중요한 문제는 메모리 누수입니다. 왜 이런일이 발생 할 까요? ViewPager에 의해 사용되는 Fragment 스택을 가지고 있고, `getSupportFragmentManager()`를 사용했기 때문에 모든 Fragment들이 Activity에 쌓입니다. 이제 부모 Fragment를 닫으면 부모는 닫히지만, 모든 자식 Fragment들은 활성 상태이고 아직 메모리 상에 있기 때문에 사라지지 않습니다. 따라서 누출이 발생합니다. 부모 Fragment 뿐만 아니라 모든 자식 Fragment도 힙 메모리에서 지훌 수 없기 때문에 누출됩니다. 그렇기에 Fragment에서 `getSupportFragmentManager()`를 사용하지 마세요

## DialogFragment, ChildFragment, BottomSheetFragment에서 부모 Fragment으로의 콜백

BottomSheetFragment, DialogFragment, ChildFragment를 사용할 때 직면하는 매우 일반적인 문제입니다.

예: 

자식 Fragment 추가:
```
FragmentTransaction ft = getChildFragmentManager().beginTransaction();
Fragment1Page2 fragment = new Fragment1Page2();
ft.replace(R.id.contentLayoutFragment1Page2, fragment);
ft.setTransition(FragmentTransaction.TRANSIT_FRAGMENT_FADE);
ft.addToBackStack(null);
ft.commit();
```

bottomSheetFragment의 다른 예:
```
BottomSheetDialogFragment fragment = BottomSheetDialogFragment.newInstance();
fragment.show(getChildFragmentManager(), fragment.getTag());
```

이제 이 자식 Fragment에 상위 Fragment로 콜백을 원한다고 가정합니다. 대부분의 사람들은 Activity를 사용해 두 Fragment 사이에 연결을 만들고 인터페이스 리스너를 Fragment의 매개 변수로 전달하는 사람은 거의 없습니다.(이는 실제로 피해야하는 나쁜 습관입니다.) 가장 좋은 방법은 자식 Fragment에서 `getParentFragmet()`를 호출하여 콜백을 만드는 것입니다. 이것은 매우 간단하며 아래 예를 생각해보세요.

```
dialogFragment.show(ParentFragment.this.getChildFragmentManager(), "dialog_fragment");
```

그런 다음 자식 Fragment에 다음 코드를 추가하여 콜백을 부모 Fragment로 설정하세요.

```
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    try {
        callback = (Callback) getParentFragment();
    } catch (ClassCastException e) {
        throw new ClassCastException("Calling fragment must implement Callback interface");
    }
}
```

그게 답니다. 이제 쉽게 부모 Fragment로  콜백을 줄 수 있습니다.

동일한 방법을 사용하여 ViewPager내의 하위 Fragment에서 ViewPager를 보유한 상위 Fragment로 콜백을 생성 할 수 있습니다.

## ViewPager에서 Fragment를 사용 할 때 FragmentStateAdapter와 FragmentPagerAdapter 중 어떤 것을 사용할 지


FragmentPagerAdapter는 모든 Fragment을 메모리에 저장하며 ViewPager에서 많은 Fragment를 사용하면 메모리 오버 헤드가 증가 할 수 있습니다. FragmentStatePagerAdapter는 Fragment의 savedInstanceState만 저장하고 Fragment가 포커스를 잃은 모든 Fragment를 삭제합니다. 

따라서 많은 Fragment를 사용하게 되면 FragmentStatePagerAdapter를 사용하고 ViewPager에서 사용되는 Fragment가 3개 미만이면 FragmentPagerAdapter를 사용하세요.

일반적으로 직면하는 몇 가지 문제를 살펴 보겠습니다.

### ViewPager 업데이트가 작동하지 않음

ViewPager Fragment는 FragmentManager에 의해 Fragment 또는 Activity에서 관리되며 FragmentManager는 모든 ViewPager Fragment의 인스턴스를 보유합니다.

ViewPager가 새로 고쳐지지 않는다면 FragmentManager가 이전에 생성된 오래된 Fragment를 보유하고 있는겁니다. FragmentManger가 Fragment 인스턴스를 보유한 이유를 찾아야합니다. 누출이 있습니까? 이상적으로 ViewPager를 새로 고치려면 다음 코드가 작동합니다. 그렇지 않으면 무언가 잘못된 것 입니다.

```
List<String> strings = new ArrayList<>();
strings.add("1");
strings.add("2");
strings.add("3");
viewPager.setAdapter(new PagerFragAdapter(getSupportFragmentManager(), strings));
strings.add("4");
viewPager.getAdapter().notifyDataSetChanged();
```

### ViewPager에서 현재 Fragment에 접근



이것은 또한 우리가 겪는 매우 일반적인 문제입니다. 이 문제가 발생하면 어댑터 내에 Fragment 리스트를 구현하거나 일부 태그를 사용하여 Fragment에 접근하세요. 다른 옵션도 있는데 FragmentStateAdapter와 FragmentPagerAdapter는 모두 `setPrimaryItem()`을 제공합니다. 아래와 같이 현재 Fragment를 설정하는데 사용 할 수 있습니다.

```
BlankFragment blankFragment;
@Override
public void setPrimaryItem(@NonNull ViewGroup container, int position, @NonNull Object object) {
    if (getBlankFragment() != object) {
        blankFragment = (BlankFragment) object;
    }
    super.setPrimaryItem(container, position, object);
}
public BlankFragment getBlankFragment() {
    return blankFragment;
}
```

모든 사람들이 더 잘 이해할 수 있도록이 간단한 ViewPager 프로젝트를 [GitHub](https://github.com/amodkanthe/ViewPagerTest)에 남겨두고 있습니다.

## FragmentTransaction Add vs Replace

Activity에는 Fragment가 표시되는 컨테이너가 있습니다. 

*Add*는 단순히 컨테이너에 Fragment를 추가합니다. FragmentA 및 FragmentB를 컨테이너에 추가한다고 가정하십시오. 컨테이너에는 FragmentA 및 FragmentB가 있으며 컨테이너가 FrameLayout인 경우 Fragment는 다른 Fragment 위에 추가됩니다.

*Replace*는 단순히 컨테이너 상단의 Fragment를 교체하므로 FragmentC 생성을 현재 상단에 있는 FragmentB 변경하면 FragmentB가 컨테이너에서 제거되고(`addToBackStack()`를 호출하지 않는 이상) FragmentC가 맨 위에 나타납니다.

언제 어떤 것을 사용해야합니까? `replace`는 현재 Fragment를 제거하고 새로운 Fragment를 추가합니다. 이는 뒤로가기 버튼을 누르면 대체된 Fragemnt의 `onCreateView()`가 호출되어 생성됩니다. 반면에 `add`는 기존 Fragment을 유지하고 새로운 Fragment을 추가하여 기존 Fragment이 활성화되고 '일시 중지'상태가 아님을 의미합니다. 따라서 `onCreateView()`에서 뒤로가기 버튼을 누르때 기존 Fragment (새 Fragment이 추가되기 전에 있던 Fragment) `onCreateView()`는 호출되지 않습니다. Fragment의 lifecycle 이벤트와 관련하여 `onPause()`, `onResume()`, `onCreateView()` 및 기타 lifecycle 이벤트는 `replace`의 경우 호출되지만 `add`의 경우 호출되지 않습니다.

현재 Fragment을 다시 방문 할 필요가 없고 현재 Fragment가 더 이상 필요하지 않은 경우 Fragment `replace`를 사용하세요. 또한 앱에 메모리 제한이 있는 경우 `add` 대신 `replace`를 사용하세요.


## Fragment Receivers, Broadcasts와 Memory Leaks

Fragment 내에서 receiver를 사용할 때 일반적인 실수는 `onPause()` 또는 `onDestory()`에서 receiver의 해제를 잊는 것입니다. `onCreate()` 또는 `onResume()`내에서 receiver를 처리하기 위해 Fragment를 등록하는 경우 `onPause()` 또는 `onDestory()` 내에서 Fragment를 등록 해제 해야합니다.

```
LocalBroadcastManager.getInstance(getActivity()).unregisterReceiver(mYourBroadcastReceiver);
```

또한 Broadcast receiver를 수신하는 여러 Fragment가 있는 경우 `onResume()`에 등록하고 `onPause()`에서 등록을 취소하세요. `onCreate()` 및 `onDestroy()`에서 등록 및 등록 취소를 하면 Fragment의 `onDestroy()`가 호출되지 않을 수 있으므로 다른 Fragment가 브로드 캐스트를 수신하지 않습니다.

## Fragment BottomBarNavigation와 NavigationDrawer 다루는 방법

BottomBarNavigation 및 NavigationDrawer를 사용하는 경우 Fragment를 다시 만들거나 같은 Fragment를 여러 번 추가하는 것과 같은 문제가 자주 발생합니다. 

이러한 경우 `add`나 `replace` 대신 fragment transaction의 `show` 및 `hide`를 사용할 수 있습니다.

네비게이션을 관리하고 FragNav라는 Fragment의 재생을 피하는 아름다운 라이브러리도 있습니다.

[링크](https://github.com/ncapdevi/FragNav)

## commit()와 commitAllowingStateLoss()

Activity가 resume 상태가 아닌데 Fragment를 커밋하려고 하면 앱이 중단됩니다. 이를 피하려면 Activity 또는 Fragment resume 상태에 있는지 또는 `isAdded()` / `isResumed()`가 아닌지 체크해야합니다.

Fragment의 상태에 대해 신경쓰지 않는다면 다른 해결책으로 `commitAllowingStateLoss()`를 호출 할 수 있다는 것입니다. 이를 통해 Activity가 종료되었거나 resume 상태가 아니여도 Fragment를 추가하거나 교체 할 수 있습니다.

## Fragment Option Menus

Fragment 내부에서 옵션 메뉴를 사용 할  때는 다음 행을 추가해야합니다. 사람들은 종종 이것을 추가하는 것을 잊어 버리고 툴바에서 옵션이 어디에 있는지 궁금해 해습니다.

```
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setHasOptionsMenu(true);
}
```

Fragment 내에서 툴바를 사용할 때 코드를 사용하여 메뉴를 inflate 할 수 있습니다.

```
getToolbar().inflateMenu(R.menu.toolbar_menu_gmr);
```

또는 `createOptionsMenu()`를 재정의 할 수 있지만 Super class에 의존하지 않으므로 위 방법을 선호합니다.

## Fragment getActivity(), getView() NullPointers Exceptions

백그라운드 프로세스가 결과를 게시하고 Fragment가 스택 또는 resume 상태가 아닌 경우 Fragment view에 접근하면 NullPointerException이 발생합니다. 따라서 백그라운드 작업 또는 지연 후 `getView()` 또는 `getActivity()`에 접근 할 때마다 종료 시 모든 백그라운드 작업을 취소해야합니다.

```
new Handler().postDelayed(new Runnable() {
    @Override
    public void run() {
        if(getActivity() != null && getView() != null && isAdded) {
            PagerFragAdapter pagerFragAdapter = (PagerFragAdapter) viewPager.getAdapter();
            pagerFragAdapter.getBlankFragment().setLabel();
        }
    }
}, 500);
```

## Nested Fragment onActivityResult

중첩된 Fragment에서 `onActivityResult()`는 호출되지 않습니다.

Android 지원 라이브러리에서 `onActivityResult()`의 호출 순서는 다음과 같습니다.

1. `Activity.dispatchActivityResult()`
2. `FragmentActivity.onActivityResult()`
3. `Fragment.onActivityResult()`

부모 Fragment나 Activity에서 `onActivityResult()`를 재정의하고 중첩된 Fragment로 값을 전달 해야합니다.

```
@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
	List<Fragment> fragments = getChildFragmentManager().getFragments();
    if (fragments != null) {
        for (Fragment fragment : fragments) {
            fragment.onActivityResult(requestCode, resultCode, data);
        }
    }
}
```

## Fragment와 Bundle

Fragment에 인수를 전달 할 때마다 생성자 대신 Bundle을 사용하고 있는지 확인하세요.

[Android 문서](https://developer.android.com/reference/android/app/Fragment.html)에는 다음이 명시되어 있습니다.

*모든 Fragment는 빈 생성자가 있어야하며 Activity 상태를 복원 할 때 인스턴스화 할 수 있습니다. Fragment의 sub class는 매개 변수가 있는 다른 생성자가 없어야합니다. 이러한 생성자는 Fragment가 다시 인스턴스화 될 때 호출되지 않기 때문입니다. 대신 호출자는 `setArguments(Bundle)`을 사용하여 인수를 제공하고 나중에 `getArguments()`를 사용하여 Fragment에 인수를 검색 할 수 있습니다.*

그렇기 때문에 Bundle을 사용하여 프래그먼트의 매개 변수를 설정하는 것이 좋습니다. Fragment를 다시 인스턴스화 할 때 시스템이 값을 더 쉽게 복원 할 수 있습니다.

## Back Navigation

세부 정보화면에서 뒤로가기 버튼을 누르면 사용자는 마스터 화면으로 돌아갑니다. Transaction을 커밋하기 전에 `addToBackStack()`을 호출하세요.

```
getSupportFragmentManager().beginTransaction()
                           .add(detailFragment, "detail")
                           // Add this transaction to the back stack
                           .addToBackStack(null)
                           .commit();
```

Back stack에 FragmentTransaction 객체가 있고 사용자가 back 버튼을 누르면 FragmentManager는 가장 최근에 트랜잭션을 Back stack에서 pop하고 역 동작(예: Fragment가 추가된 경우 Fragment를 제거)을 수행합니다.

## 결론

Fragment는 처음에 매우 쉬운 것처럼 보이지만 더 많은 부분이 있습니다. Fragment를 사용하는 동안 Memory, navigation, callback, bundle과 같은 여러가지 사항에 주의해야합니다. 이 아티클에서 가장 일반적으로 직면하는 문제와 가장 일반적으로 저지른 실수가 다뤄지기를 바랍니다.
