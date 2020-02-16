---
title: "[Android] [Dagger] 시작하기"
categories:
  - Android
tags:
  - Android
  - Dagger
---

# Dagger 시작하기
Android 의 Dependency Injection(이하 DI) 라고 하면 가장 먼저 떠오르는건 당연코 Dagger 이다.

Dagger 는 Square 에서 개발하다가 현재는 Google 에서 관리하고 있는 오픈소스 Depedency Injector framework 이다. 


최근엔 Kotlin 을 많이 쓰게 되면서 Koin 도 각광을 받고 있지만 구관이 명관이라고 했던가, 여전히 Dagger 는 DI 를 적용할려는 안드로이드 개발자 사이에서는 1순위 이다.

그리고 Google IO 2019 에서도 Dagger 관련 세션이 있었는데, 이를 통해 Google 내부에서도 Dagger 를 활발히 사용하고 있고 어떻게 발전시킬지 고민을 많이 하고 있는 듯 하다.


Dagger 에 앞서 DI 가 무엇인지 DI 를 사용하면 뭐가 좋은지 등을 얘기하면 참 좋겠지만, 아직 썰을 풀기에는 내공이 부족한지라 우선 간단한 적용기를 써보고자 한다.


개인 프로젝트에 적용하기 위해 시도했던 내용들을 글로 푸는지라 코드는 Github 에 공개하지 않았다. 

하지만 대부분의 (실제로 빌드 가능하고 동작하는) 코드를 아래에 대부분 남길 예정이기 때문에 적용하는데 참고하면 좋을 것 같다.

# Dagger 초간단 적용하기

## 의존 추가하기
Dagger 를 사용하기 위해선 우선 Dagger 를 프로젝트에 추가해야한다.

아래 내용을 app 모듈의 build.gradle 의 `dependencies` 에 추가하자.

(2020년 2월 15일 기준, 최신 버전 2.26을 적용했다. 버전을 번경하고 싶다면 모든 버전을 통일해서 변경하면 된다.)

{% highlight gradle linenos %}
dependencies {
    implementation 'com.google.dagger:dagger:2.26'
    implementation 'com.google.dagger:dagger-android:2.26'
    implementation 'com.google.dagger:dagger-android-support:2.26'

    kapt 'com.google.dagger:dagger-compiler:2.26'
    kapt 'com.google.dagger:dagger-android-processor:2.26'
}
{% endhighlight %}

만약 프로젝트에 kapt 가 적용되어있지 않다면 app 모듈의 build.gradle 상단에 아래 내용도 추가하도록 하자.

{% highlight gradle linenos %}
apply plugin: 'kotlin-kapt'
{% endhighlight %}

의존이 잘 추가되었는지 빌드를 해보자. 빌드에 성공했다면 의존이 제대로 추가되었다고 볼 수 있다.

## Application 추가
Application 을 추가해보자. 기존 Application 과 다른 점이라면 `DaggerApplication` 을 상속한다는 점이다.

{% highlight kotlin linenos %}
class App : DaggerApplication() {
    override fun applicationInjector(): AndroidInjector<out DaggerApplication>? {
      // TODO 구현해야함
    }
}
{% endhighlight %}

그리고 `AndroidManifest.xml` 에 아래 속성을 application 태그에 추가한다.

{% highlight xml linenos %}
<application
    android:name=".App">
    <!--이하 생략-->
</application>
{% endhighlight %}

아직 `fun applicationInjector()` 메소드가 구현되어있지 않기 때문에 빌드가 되지 않는다. `ApplicationComponent` 를 작성해보자.

## ApplicationComponent, ApplicationModule 추가
사실상 루트 컴포넌트인 `ApplicationComponent` 와 `ApplicationModule` 을 작성해보자.

{% highlight kotlin linenos %}
@Singleton
@Component(
    modules = [
        AndroidSupportInjectionModule::class,
        ApplicationModule::class
    ]
)
interface ApplicationComponent : AndroidInjector<App> {
    @Component.Builder
    interface Builder {
        @BindsInstance
        fun application(application: Application): Builder

        fun build(): ApplicationComponent
    }
}
{% endhighlight %}

{% highlight kotlin linenos %}
@Module
class ApplicationModule {
    @Provides
    fun providesContext(application: Application): Context {
        return application
    }
}
{% endhighlight %}


그럼 이제 프로젝트를 한번 빌드해보자. 그럼 우리가 작성한 `ApplicationComponent` 을 활용해서 `DaggerApplicationComponent` 을 생성해준다.

그럼 이를 바탕으로 `App` 을 수정해보자.

{% highlight kotlin linenos %}
class App : DaggerApplication() {
    override fun applicationInjector(): AndroidInjector<out DaggerApplication>? =
        DaggerApplicationComponent.builder()
            .application(this)
            .build()
}
{% endhighlight %}

## ActivityModule 추가
Activity 에 대한 모듈인 `ActivityModule` 을 추가해보자.

{% highlight kotlin linenos %}
@Module
abstract class ActivityModule {
    @ContributesAndroidInjector
    abstract fun bindMainActivity(): MainActivity
}
{% endhighlight %}

`ApplicationComponent` 의 `@Component` 어노테이션의 modules 속성에 `ActivityModule` 을 추가하자

{% highlight kotlin linenos %}
@Singleton
@Component(
    modules = [
        AndroidSupportInjectionModule::class,
        ApplicationModule::class,
        ActivityModule::class
    ]
)
interface ApplicationComponent : AndroidInjector<TomorrowardApplication> {
  // 생략
}
{% endhighlight %}

## MainActivity 변경
MainActivity 의 부모를 `DaggerAppCompatActivity` 로 변경하자. 이로서 새 프로젝트에 Dagger 적용이 완료되었다.

빌드해서 실행해보고 크래시가 발생하지 않으면 정상동작한 것이다.

{% highlight kotlin linenos %}
class MainActivity : DaggerAppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
{% endhighlight %}

# DI 동작확인을 위한 Inject 테스트
빌드에 성공하고 크래시도 발생하지 않지만, 그래도 DI가 진짜 동작하는지 확인은 해보고 싶다.

간단하게 토스트를 찍는 코드를 작성해서 실제로 Inject 가 되는지 확인해보자.

테스트를 위한 코드이다. 실제 프로젝트에서 이딴식으로(?!) 구현하지는 말자.

{% highlight kotlin linenos %}
@Module
class ToastModule {
    @Provides
    fun toastService(): ToastService = ToastServiceImpl()
}

interface ToastService {
    fun showToast(context: Context, msg: String)
}

class ToastServiceImpl : ToastService {
    override fun showToast(context: Context, msg: String) =
        Toast.makeText(context, msg, Toast.LENGTH_SHORT).show()
}
{% endhighlight %}

`ApplicationComponent` 의 `@Component` 어노테이션의 modules 에 `ToastModule` 을 추가한다.

{% highlight kotlin linenos %}
@Singleton
@Component(
    modules = [
        AndroidSupportInjectionModule::class,
        ApplicationModule::class,
        ActivityModule::class,
        ToastModule::class // ToastModule 추가
    ]
)
interface ApplicationComponent : AndroidInjector<App> {
  // 생략
}
{% endhighlight %}

`MainActivity` 에 Inject 시켜보자.

{% highlight kotlin linenos %}
class MainActivity : DaggerAppCompatActivity() {
    @Inject
    internal lateinit var toastService: ToastService

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        toastService.showToast(this, "MainActivity::onCreate")
    }
}
{% endhighlight %}

실행해보면 토스트가 정상적으로 노출되는 것을 확인할 수 있다.

# 마치며
간단 적용기라고 했지만 적다보니 글이 좀 많이 길어진 것 같다. 하지만 실제로 적용해보면 수 분만에 모두 끝낼 수 있다.

적용기를 한번 알아보았다. 러닝커브가 있다고는 하지만 적용이 생각보다 복잡하지는 않다. 신규 프로젝트를 빌드업할 때는 겁내지말고 Dagger 를 적용하고 시작해보자.