---
title: "Jetpack Navigation 에서 Fragment 를 show/hide 하는 방법"
categories:
  - Android
tags:
  - Android
  - Jetpack
  - Navigation
toc: true
---

## 문제
### 하단 탭 구현

요즘의 앱들은 대부분 하단 탭이 있다. Android 에서는 Material 라이브러리에 있는 `BottomNavigationView` 를 이용해서 구현한다.

직관적인 구현으로는 `Fragment` 들을 replace 하면 될 것으로 보이지만, 실제로는 그렇지 않은 경우가 있다.

탭 전환시 이전 탭의 화면이 저장되기를 원한다면 replace 로는 구현할 수 없다. 이 경우, show/hide 를 이용해서 구현한다.

### Jetpack Navigation 에서는?

`Jetpack Navigation`(이하 Navigation) 을 사용하면 아주 간편하게 `BottomNavigationView` 에 `NavController` 를 붙일 수 있어서 실제 Fragment 제어를 할 필요가 없어진다.

하지만 기본 동작(`FragmentNavigator`)이 replace 이기 때문에 show/hide 를 쓰고 싶은 경우에는 사용할 수 없다.

필자의 경우 탭이 있는 시작화면에 `NavHostFragment` 를 두고 싶었는데, 그러기 위해선 `BottomNavigationView` 에 `NavController` 를 붙일 수 밖에 없었다.

그럼 Navigation 에서 show/hide 를 사용하고 싶은 경우, 어떻게 해야하는걸까?

## Navigator

Navigation 에는 `Navigator` 들이 정의되어있다. 우리가 Navigation XML 에서 `<fragment ...>` 이렇게 정의할 수 있는 이유는, `fragment` 이름을 제어할 수 있는 `FragmentNavigator` 가 있기 때문이다. 그리고 `FragmentNavigator` 의 기본 동작은 replace 이다.

{% highlight java linenos %}
@Navigator.Name("fragment")
public class FragmentNavigator extends Navigator<FragmentNavigator.Destination> {
  @Nullable
  @Override
  public NavDestination navigate(@NonNull Destination destination, @Nullable Bundle args,
          @Nullable NavOptions navOptions, @Nullable Navigator.Extras navigatorExtras) {
    // ...
    ft.replace(mContainerId, frag);
    // ...
  }
  // ...
}
{% endhighlight %}

이미 `fragment` 에 대해서 `Navigator` 가 정의되어 있기 때문에, 우리는 따로 `Navigator` 를 정의해볼까 한다.

### Custom Navigator
`Navigator` 는 실제로 이동을 하고 백스택을 관리하는 로직을 담고 있다. 당연한 이야기일 수도 있지만, Activity, Fragment, DialogFragment 로 이동하는 방식은 다르다. 그런 의미로 `Navigator` 가 모두 따로 정의되어있다.

실제로 Fragment 를 제어할 예정이므로 `FragmentNavigator` 를 참고해서 구현하면 된다. 핵심 코드는 [`navigate`](https://developer.android.com/reference/androidx/navigation/Navigator#navigate(androidx.navigation.NavDestination,android.os.Bundle,androidx.navigation.NavOptions,androidx.navigation.Navigator.Extras)) 구현과 [`NavDestination`](https://developer.android.com/reference/androidx/navigation/NavDestination) 구현이다.

### NavDestination 구현
`NavDestination` 은 실제 navigate 가 되었을 때 이동될 노드의 정보가 담겨져온다. 우리는 기본적인 정보외에 `Fragment` 정보도 함꼐 담아야한다. 이를 위해 `FragmentNavigator` 구현과 동일하게 navigation 리소스로부터 `Fragment` 정보를 함께 담는다.

{% highlight kotlin linenos %}
@NavDestination.ClassType(Fragment::class)
class Destination(navigator: BottomNavigator) : NavDestination(navigator) {
    internal var className: String? = null
        private set

    override fun onInflate(context: Context, attrs: AttributeSet) {
        super.onInflate(context, attrs)

        className = context.resources.obtainAttributes(attrs, R.styleable.BottomNavigator)
            .use {
                it.getString(R.styleable.BottomNavigator_android_name)
            }
    }
}
{% endhighlight %}

그리고 `Navigator` 를 정의하고 `createDestination` 를 통해 객체를 생성하게 한다.

{% highlight kotlin linenos %}
@Navigator.Name("bottom")
class BottomNavigator(
    @IdRes private val fragmentContainerId: Int,
    private val fragmentManager: FragmentManager
) : Navigator<BottomNavigator.Destination>() {
    private val backStack: Deque<String> = ArrayDeque()

    override fun createDestination(): Destination = Destination(this)

    // ...
}
{% endhighlight %}

### 탐색, 그리고 백스택
이제 실제 이동하는 로직을 구현하면 된다. 이는 `Fragment` 를 제어할 줄 알면 크게 어렵지 않다. 클래스 이름으로부터 `Fragment` 를 생성한다는 것외에는 기본적인 `Fragment` 제어와 동일하다.

{% highlight kotlin linenos %}
override fun navigate(
    destination: Destination,
    args: Bundle?,
    navOptions: NavOptions?,
    navigatorExtras: Extras?
): NavDestination? {
    val className = destination.className ?: return null
    val tag = className.split('.').last()

    if (backStack.peekLast() == tag) {
        return null
    }

    if (backStack.peekLast() != tag) {
        backStack.addLast(tag)
    }

    val current = fragmentManager.findFragmentByTag(tag)
    fragmentManager.commit {
        if (current == null) {
            val fragment = fragmentManager.fragmentFactory.instantiate(
                ClassLoader.getSystemClassLoader(),
                className
            )
            add(fragmentContainerId, fragment, tag)
        } else {
            show(current)
        }

        hideOthers(tag)
    }

    return destination
}
{% endhighlight %}

탐색은 구현이 완료되었고, 이제 백스택 구현이 필요하다. `Fragment` 하나만 show 되고 나머지는 모두 hide 하고 있으니 백스택은 별도로 관리해보자.

{% highlight kotlin linenos %}
private val backStack: Deque<String> = ArrayDeque()

override fun popBackStack(): Boolean {
    val tag = backStack.pollLast() ?: return true
    val newCurrentTag = backStack.peekLast() ?: return true
    val newCurrent = fragmentManager.findFragmentByTag(newCurrentTag)
    fragmentManager.commit {
        newCurrent?.let {
            show(it)
            hideOthers(newCurrentTag)
        }
    }
    return true
}
{% endhighlight %}

### Navigator 등록하기
이제 `Navigator` 를 등록해보자. 기존에는 `Activity`의 XML에 `navGraph` 를 정의했었다. 이제 `navGraph` 가 로드되기 전에 `Navigator` 를 등록해야하므로, XML에 `navGraph` 정의는 하지말고 아래와 같이 코드를 작성하자.

{% highlight kotlin linenos %}
navController.apply {
    navigatorProvider.addNavigator(
        BottomNavigator(
            R.id.fragment_container,
            supportFragmentManager
        )
    )
    // set a graph at code not XML, because add a custom navigator
    setGraph(R.navigation.bottom_navigation)

    binding.bottomNavigation.setupWithNavController(this)
}
{% endhighlight %}

### XML에서의 사용
이제 Navigation XML 을 정의해보자. `fragment` 태그 대신 `Navigator` 정의시 달아둔 어노테이션의 이름으로 태그를 달아보자.

필자는 `@Navigator.Name("bottom")` 으로 정의했으므로 `<bottom>` 태그를 사용했다.

{% highlight xml linenos %}
<bottom
    android:id="@+id/home_menu"
    android:name="io.viewpoint.moviedatabase.ui.home.HomeFragment"
    android:label="HomeFragment"
    tools:layout="@layout/fragment_home" />
{% endhighlight %}

{% capture notice-warning %}
`BottomNavigationView` 을 위해서 정의한 menu 의 `android:id` 와 Navigation 의 `android:id` 는 동일해아한다.

예)
`menu.xml`
{% highlight xml linenos %}
<item
    android:id="@+id/home_menu"
    android:icon="@drawable/ic_home"
    android:title="@string/menu_home" />
{% endhighlight %}

`navigation.xml`
{% highlight xml linenos %}
<bottom
    android:id="@+id/home_menu"
    android:name="io.viewpoint.moviedatabase.ui.home.HomeFragment"
    android:label="HomeFragment"
    tools:layout="@layout/fragment_home" />
{% endhighlight %}
{% endcapture %}

<div class="notice--warning">
  <h4 class="no_toc">주의:</h4>
  {{ notice-warning | markdownify }}
</div>

## 백스택의 기본 동작에 대하여
`BottomNavgationView` 에 `setupWithNavController` 를 통해 `NavController` 를 설정하게 된다. 이 때, 백스택의 기본 동작은 첫 탭으로 이동하는 것이다.

즉, `1 -> 2 -> 3 -> ...` 로 이동하면 백스택이 쌓일 것 같지만 실제로는 그렇지 않다. 여기서 백키를 누르면 최초 진입점이었던 `1` 로 이동하게 된다.

만약 모든 탭 이동을 백스택에 쌓고 싶으면, 메뉴 XML 에서 모든 메뉴에 `android:menuCategory="secondary"` 속성을 부여하자. 그러면 `NavigationUI` 의 아래 코드가 무시되면서 백스택이 모두 쌓이게 된다.

{% highlight java linenos %}
// NavigationUI.java
public static boolean onNavDestinationSelected(@NonNull MenuItem item,
        @NonNull NavController navController) {
    // ...
    if ((item.getOrder() & Menu.CATEGORY_SECONDARY) == 0) {
        builder.setPopUpTo(findStartDestination(navController.getGraph()).getId(), false);
    }
    // ...
}
{% endhighlight %}

## 결론
위와 같이 replace 대신 show/hide 를 이용한 `Fragment` 제어를 `Navigatior` 를 구현하는 것으로 만들어보았다.

예제로 쓰인 Navigator 의 구현은 [이 곳](https://github.com/hide1202/MovieDatabase/blob/develop/app/src/main/java/io/viewpoint/moviedatabase/platform/navigation/BottomNavigator.kt) 에서 볼 수 있고, 구현이 포함된 프로젝트 리포지토리는 [이 곳](https://github.com/hide1202/MovieDatabase) 을 참고하면 된다.

실제로 리포지토리에 들어가서 보면 재생성시 대응을 위해 `onSaveState`, `onRestoreState` 구현도 포함되어있으니 참고하면 좋을 것 같다.

## 참조
- Navigation: [https://developer.android.com/guide/navigation](https://developer.android.com/guide/navigation?hl=ko)
- MovieDatabase: [https://github.com/hide1202/MovieDatabase](https://github.com/hide1202/MovieDatabase)