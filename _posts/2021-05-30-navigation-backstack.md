---
title: "`BottomNavigationView` + `Jetpack Navigation` 에서의 백스택"
categories:
  - Android
tags:
  - Android
  - Jetpack
  - Navigation
original_backstack:
  - url: /assets/images/posts/navigation-backstack/original-backstack.gif
    image_path: /assets/images/posts/navigation-backstack/original-backstack.gif
    alt: "Original-BackStack"
secondary_backstack:
  - url: /assets/images/posts/navigation-backstack/secondary-backstack.gif
    image_path: /assets/images/posts/navigation-backstack/secondary-backstack.gif
    alt: "Secondary-BackStack"
toc: true
---

## 하단 탭에서의 백스택
하단 탭이 있는 앱의 경우, 탭 이동에 다양한 백스택 전략을 사용할 수 있다. 가장 기본적으로는 백스택에 쌓지 않고 바로 닫는 경우가 있다.

그리고 탭 이동마다 백스택에 쌓고 백 이동시 계속 이전 탭으로 이동시키고, 마지막에는 앱을 닫게할 수도 있다.

[앞선 글](../bottom-navigator/)에서 `BottomNavigationView` 에 `Jetpack Navigation` 을 적용하는 방법에 대해서 알아보았다. 해당 글에서도 언급은 했지만 `BottomNavigationView` + `Jetpack Navigation` 조합을 사용할 경우 백스택 전략은 크게 두 가지로 볼 수 있다.


## 백스택 전략
### NavigationUI 의 기본 설정
`BottomNavigationView` 에 `NavController` 를 설정할 때 `NavigationUI.setupWithNavController` 를 사용한다. 이를 사용하면 아래 내부 구현에 따라 이동 전에 첫 시작점으로 pop 된다.

{% highlight java linenos %}
if ((item.getOrder() & Menu.CATEGORY_SECONDARY) == 0) {
    builder.setPopUpTo(findStartDestination(navController.getGraph()).getId(), false);
}
{% endhighlight %}

실제 동작 화면은 아래와 같다.

{% include gallery id="original_backstack" %}

### 모든 탭 선택을 백스택으로!
모든 탭 이동을 백스택에 쌓고 싶으면, 메뉴 XML 에서 모든 메뉴에 `android:menuCategory="secondary"` 속성을 부여하자. 그러면 위 `NavigationUI` 의 if 문에 의해 `setPopUpTo` 로직이 수행되지 않는다.

{% highlight xml linenos %}
<item
    android:id="@+id/home_menu"
    android:icon="@drawable/ic_home"
    android:menuCategory="secondary" 
    android:title="@string/menu_home" />
{% endhighlight %}

실제 동작 화면은 아래와 같다.

{% include gallery id="secondary_backstack" %}

## 결론
사실 아직 백스택에 아무것도 안 쌓고 바로 앱을 닫는 방법은 파악하지 못했다. 이 부분은 차후에 방법을 찾게되면 별도로 포스팅을 해야겠다.

## 참조
- Navigation: [https://developer.android.com/guide/navigation](https://developer.android.com/guide/navigation?hl=ko)
- MovieDatabase: [https://github.com/hide1202/MovieDatabase](https://github.com/hide1202/MovieDatabase)