---
layout: post
category : Android
title : "[Sample] -MVP,Dagger,RxJava,Retrofit을 사용한 간단한 Android App"
description : ""
tags : [Android, Sample]
---

{% include JB/setup %}

> *이 문서는 [A Simple Android Apps with MVP, Dagger, RxJava, and Retrofit](https://medium.com/@nurrohman/a-simple-android-apps-with-mvp-dagger-rxjava-and-retrofit-4edb214a66d7#.evb84ghvt)을 번역한 문서입니다.*

![](https://cdn-images-1.medium.com/max/800/1*7nit2zDwYQNV9guqd0bkuw.png)

얼마전 Clean 아키텍처로 Android 앱을 개발하는 방법에 대해서 학습 했습니다. 샘플 프로젝트에서 MVP, Dagger, RxJava, Retrofit을 구현하는 방법을 배웠습니다. Mock 서버에서 API를 호출하고 텍스트와 이미지를 RecycleView로 가져오는 Android 앱을 만들었습니다. 이제 이 앱을 개발하는 방법은 단계 별로 공유하려합니다.

연습하기 전에 MVP, Dagger, RxJava 및 Retrofit이 무엇인지 이해를 해야합니다. 아래는 MVP, Dagger, RxJava 및 Retrofit에 대한 간략한 설명입니다.

### MVP

MVP(Model View Presenter)는 로직과 프리젠테이션 레이어를 분리 할 수 있는 패턴으로, 인터페이스 작동 방식은 화면에서 표현되는 것과 구분됩니다. 이상적으로 MVP 패턴은 비지니스 로직과 뷰가 분리되어있는 것입니다.

### Dagger

대거는 의존성 주입기입니다. [이 사이트](http://square.github.io/dagger/)에서 Dagger에 대한 자세한 내용을 볼 수 있습니다.

### RxJava

RxJava는 [Reactive Extensions](http://reactivex.io/)의 Java 버전입니다. Observable Sequences를 사용하여 비동기 및 이벤트 기반 프로그램을 작성하기위한 라이브러리입니다.

### Retrofit

[Retrofit](http://square.github.io/retrofit/)은 Android 및 Java 용 REST 클라이언트 라이브러리입니다. JSON을 검색하고 업로드하는 것이 간단하게 제공됩니다. Retrofit은 HTTP 처리에 OkHttp 라이브러리를 사용합니다.

Ok, 코드를 입력해봅시다!

먼저 새 프로젝트를 만들고 build.gradle을 열고 아래와 같이 수정합니다.

```
apply plugin: 'com.android.application'
apply plugin: 'com.neenbedankt.android-apt'

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
    }
}

android {
    ...

    defaultConfig {
        ...

        buildConfigField "int", "LIMIT", "100"
        buildConfigField "String", "BASEURL", "\"http://private-b8cf44-androidcleancode.apiary-mock.com/\""
        buildConfigField "int", "CACHETIME", "432000" // 5days
    }
    ...
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.4.0'
    compile 'com.android.support:design:23.4.0'
    compile 'com.android.support:recyclerview-v7:23.4.0'
    compile 'com.android.support:cardview-v7:23.4.0'
    compile 'com.github.bumptech.glide:glide:3.7.0'

    provided 'org.glassfish:javax.annotation:10.0-b28'

    compile 'com.google.code.gson:gson:2.6.2'
    compile 'com.squareup.retrofit2:retrofit:2.1.0'
    compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'
    compile 'com.squareup.retrofit2:converter-gson:2.1.0'
    compile 'com.squareup.retrofit2:converter-scalars:2.1.0'
    compile 'io.reactivex:rxandroid:1.2.1'
    compile 'io.reactivex:rxjava:1.1.6'

    apt 'com.google.dagger:dagger-compiler:2.2'
    compile 'com.google.dagger:dagger:2.2'
    provided 'javax.annotation:jsr250-api:1.0'
}
```

그리고 2개의 레이아웃을 만듭니다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".home.HomeActivity">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:divider="#00000000"/>

    <ProgressBar
        android:id="@+id/progress"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"/>

</RelativeLayout>
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:background="#EEEEEEEE">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <ImageView
            android:id="@+id/image"
            android:layout_width="match_parent"
            android:layout_height="240dp"
            android:scaleType="fitXY"/>
    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="bottom|center"
        android:gravity="top|center"
        android:orientation="vertical"
        android:background="#5000">

        <TextView
            android:id="@+id/city"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Yogyakarta"
            android:layout_gravity="center"
            android:layout_marginTop="80dp"
            android:gravity="top|center"
            android:textSize="18dp"
            android:textColor="@android:color/white"/>

        <TextView
            android:id="@+id/hotel"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="338 Hotel"
            android:layout_gravity="center"
            android:layout_marginTop="10dp"
            android:gravity="top|center"
            android:textSize="16dp"
            android:textColor="@android:color/white"/>
    </LinearLayout>
</FrameLayout>
```

다음과 같이 프로젝트를 구성하세요.

![](https://cdn-images-1.medium.com/max/800/1*zJVQc5hbIXaW1w1QRDUXNQ.png)

아래와 같이 POJO 클래스를 생성합니다.

```java
@Generated("org.jsonschema2pojo")
public class CityListData {

    @SerializedName("id")
    @Expose
    private String id;

    @SerializedName("name")
    @Expose
    private String name;

    @SerializedName("description")
    @Expose
    private String description;

    @SerializedName("background")
    @Expose
    private String background;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public String getBackground() {
        return background;
    }

    public void setBackground(String background) {
        this.background = background;
    }
}
```

```java
@Generated("org.jsonschema2pojo")
public class CityListResponse {

    @SerializedName("data")
    @Expose
    private List<CityListData> data = new ArrayList<CityListData>();

    @SerializedName("message")
    @Expose
    private String message;

    @SerializedName("status")
    @Expose
    private int status;

    public List<CityListData> getData() {
        return data;
    }

    public void setData(List<CityListData> data) {
        this.data = data;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public int getStatus() {
        return status;
    }

    public void setStatus(int status) {
        this.status = status;
    }
}
```

그런 다음 Mock 서버에서 API를 호출하는 데 사용할 HTTP 서비스를 만듭니다.

```java
public interface NetworkService {

    @GET("v1/city")
    Observable<CityListResponse> getCityList();
}
```

subscriber를 실행할 클래스를 만듭니다.

```java
public class Service {
    private final NetworkService networkService;

    public Service(NetworkService networkService) {
        this.networkService = networkService;
    }

    public Subscription getCityList(final GetCityListCallback callback) {
        return networkService.getCityList()
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .onErrorResumeNext(new Func1<Throwable, Observable<? extends CityListResponse>>() {
                    @Override
                    public Observable<? extends CityListResponse> call(Throwable throwable) {
                        return Observable.error(throwable);
                    }
                })
                .subscribe(new Subscriber<CityListResponse>() {
                    @Override
                    public void onCompleted() { }

                    @Override
                    public void onError(Throwable e) {
                        callback.onError(new NetworkError(e));
                    }

                    @Override
                    public void onNext(CityListResponse cityListResponse) {
                        callback.onSuccess(cityListResponse);
                    }
                });
    }

    public interface GetCityListCallback{
        void onSuccess(CityListResponse cityListResponse);
        void onError(NetworkError networkError);
    }
}
```

Retrofit 빌더가 포함 된 네트워크 모듈을 구현하고 `NetworkService` 클래스 및 `Service` 클래스를 제공합니다.

```java
@Module
public class NetworkModule {
    File cacheFile;

    public NetworkModule(File cacheFile) {
        this.cacheFile = cacheFile;
    }

    @Provides
    @Singleton
    Retrofit provideCall() {
        Cache cache = null;
        try {
            cache = new Cache(cacheFile, 10 * 1024 * 1024);
        } catch (Exception e) {
            e.printStackTrace();
        }

        OkHttpClient okHttpClient = new OkHttpClient.Builder()
                .addInterceptor(new Interceptor() {
                    @Override
                    public okhttp3.Response intercept(Chain chain) throws IOException {
                        Request original = chain.request();

                        // Customize the request
                        Request request = original.newBuilder()
                                .header("Content-Type", "application/json")
                                .removeHeader("Pragma")
                                .header("Cache-Control", String.format("max-age=%d", BuildConfig.CACHETIME))
                                .build();

                        okhttp3.Response response = chain.proceed(request);
                        response.cacheResponse();
                        // Customize or return the response
                        return response;
                    }
                })
                .cache(cache)
                .build();

        return new Retrofit.Builder()
                .baseUrl(BuildConfig.BASEURL)
                .client(okHttpClient)
                .addConverterFactory(GsonConverterFactory.create())
                .addConverterFactory(ScalarsConverterFactory.create())
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .build();
    }

    @Provides
    @Singleton
    @SuppressWarnings("unused")
    public NetworkService providesNetworkService(Retrofit retrofit) {
        return retrofit.create(NetworkService.class);
    }

    @Provides
    @Singleton
    @SuppressWarnings("unused")
    public Service providesService(NetworkService networkService) {
        return new Service(networkService);
    }
}
```

`@Modules`와 `@Inject`를 연결하는 인터페이스를 만듭니다.

```java
@Singleton
@Component(modules = {NetworkModule.class,})
public interface Deps {
    void inject(HomeActivity homeActivity);
}
```

`MainActivity`가 상속 받을 `BaseApp` 클래스에 Dagger 빌더를 추가합니다.

```java
public class BaseApp  extends AppCompatActivity{
    Deps deps;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        File cacheFile = new File(getCacheDir(), "responses");
        deps = DaggerDeps.builder().networkModule(new NetworkModule(cacheFile)).build();
    }

    public Deps getDeps() {
        return deps;
    }
}
```
여기까지는 어떤 Activity를 만들든 재사용 할 수 있는 클래스를 구현했습니다. 그리고 이제 HomeActivity에 대한 presenter, view, adapter를 구현합니다.

```java
public interface HomeView {
    void showWait();
    void removeWait();
    void onFailure(String appErrorMessage);
    void getityListSuccess(CityListResponse cityListResponse);
}
```

```java
public class HomePresenter {
    private final Service service;
    private final HomeView view;
    private CompositeSubscription subscriptions;

    public HomePresenter(Service service, HomeView view) {
        this.service = service;
        this.view = view;
        this.subscriptions = new CompositeSubscription();
    }

    public void getCityList() {
        view.showWait();

        Subscription subscription = service.getCityList(new Service.GetCityListCallback() {
            @Override
            public void onSuccess(CityListResponse cityListResponse) {
                view.removeWait();
                view.getityListSuccess(cityListResponse);
            }

            @Override
            public void onError(NetworkError networkError) {
                view.removeWait();
                view.onFailure(networkError.getAppErrorMessage());
            }

        });

        subscriptions.add(subscription);
    }

    public void onStop() {
        subscriptions.unsubscribe();
    }
}
```

```java
public class HomeAdapter extends RecyclerView.Adapter<HomeAdapter.ViewHolder> {
    private final OnItemClickListener listener;
    private List<CityListData> data;
    private Context context;

    public HomeAdapter(Context context, List<CityListData> data, OnItemClickListener listener) {
        this.data = data;
        this.listener = listener;
        this.context = context;
    }

    @Override
    public HomeAdapter.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_home, null);
        view.setLayoutParams(new RecyclerView.LayoutParams(RecyclerView.LayoutParams.MATCH_PARENT, RecyclerView.LayoutParams.WRAP_CONTENT));
        return new ViewHolder(view);
    }

    @Override
    public void onBindViewHolder(HomeAdapter.ViewHolder holder, int position) {
        holder.click(data.get(position), listener);
        holder.tvCity.setText(data.get(position).getName());
        holder.tvDesc.setText(data.get(position).getDescription());

        String images = data.get(position).getBackground();

        Glide.with(context)
                .load(images)
                .diskCacheStrategy(DiskCacheStrategy.SOURCE)
                .skipMemoryCache(true)
                .into(holder.background);
    }

    @Override
    public int getItemCount() {
        return data.size();
    }

    public interface OnItemClickListener {
        void onClick(CityListData Item);
    }

    public class ViewHolder extends RecyclerView.ViewHolder {
        TextView tvCity, tvDesc;
        ImageView background;

        public ViewHolder(View itemView) {
            super(itemView);
            tvCity = (TextView) itemView.findViewById(R.id.city);
            tvDesc = (TextView) itemView.findViewById(R.id.hotel);
            background = (ImageView) itemView.findViewById(R.id.image);
        }

        public void click(final CityListData cityListData, final OnItemClickListener listener) {
            itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    listener.onClick(cityListData);
                }
            });
        }
    }
}
```

마지막으로 Activity에서 presenter, view, adapter를 처리합니다.

```java
public class HomeActivity extends BaseApp implements HomeView {

    private RecyclerView list;

    @Inject
    public Service service;

    ProgressBar progressBar;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        getDeps().inject(this);

        renderView();
        init();

        HomePresenter presenter = new HomePresenter(service, this);
        presenter.getCityList();
    }

    public void renderView(){
        setContentView(R.layout.activity_home);
        list = (RecyclerView) findViewById(R.id.list);
        progressBar = (ProgressBar) findViewById(R.id.progress);
    }

    public void init(){
        list.setLayoutManager(new LinearLayoutManager(this));
    }

    @Override
    public void showWait() {
        progressBar.setVisibility(View.VISIBLE);
    }

    @Override
    public void removeWait() {
        progressBar.setVisibility(View.GONE);
    }

    @Override
    public void onFailure(String appErrorMessage) {
        // todo dialog
    }

    @Override
    public void getityListSuccess(CityListResponse cityListResponse) {
        HomeAdapter adapter = new HomeAdapter(getApplicationContext(), cityListResponse.getData(),
                new HomeAdapter.OnItemClickListener() {
                    @Override
                    public void onClick(CityListData Item) {
                        Toast.makeText(getApplicationContext(), Item.getName(),
                                Toast.LENGTH_LONG).show();
                    }
                });

        list.setAdapter(adapter);
    }
}
```

GitHub에 전체 소스 코드를 다운로드 할 수도 있습니다. [Clean Android Code](https://github.com/ennur/Clean-Android-Code)
