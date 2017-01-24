---
layout: post
category : Android
title : "[Dagger2] Dagger2 소개(Android에서 Dependency Injection 사용하기) Part2"
description : ""
tags : [Android, DI]
---

{% include JB/setup %}


> *이 문서는 [Introduction to Dagger 2, Using Dependency Injection in Android: Part 2](https://blog.mindorks.com/introduction-to-dagger-2-using-dependency-injection-in-android-part-2-b55857911bcd#.pw6urplsf)를 번역한 문서입니다.*

이 문서는 Dependency Injection의 Part 2입니다. Part 1에서는 Dependency Injection의 필요성과 이점을 이해했습니다. Dagger2에 대한 개요도 있습니다. 이 부분에서는 Android 앱에서 Dagger2를 사용하여 DI를 구현하는데 초점을 맞춥니다.

[여기서](/2017-01-18-Dagger2-소개(Android에서-Dependency-Injection-사용하기)-Part1.md) Part 1을 확인하세요.

이 튜토리얼에서는 과정을 단계적으로 분해하고 각 단계를 하나씩 분석합니다. Dagger2는 집중적인 접근 방식이 필요합니다. 그래서 적극적으로 아래의 튜토리얼을 따라 많은 질문을 하세요. 프로젝트 구조에 대해서는 아래에 언급되는 프로젝트 repo를 참조하세요.

시작합니다.

[프로젝트에 대한 GitHub 레포](https://github.com/MindorksOpenSource/android-dagger2-example)

먼저 안드로이드 앱에 대한 구조를 정의해야합니다. 코어 클래스는 다음과 같습니다.

1. `DataManager` 클래스는 앱 데이터에 대한 엑세스를 제공합니다.
2. `DbHelper` 클래스는 `DataManager`에서 `SQLite` 데이터베이스에 엑세스하는데 사용됩니다.
3. `SharedPrefsHelper`는 `DataManager`에서 `SharedPreferences`에 엑세스하는데 사용됩니다.

### Stpe 1

Android Studio에서 Empty Activity로 프로젝트를 생성하고, 앱의 `build.gradle`에 다음 dependencies를 추가합니다.

```
dependencies {
    ...
    compile "com.google.dagger:dagger:2.8"
    annotationProcessor "com.google.dagger:dagger-compiler:2.8"
    provided 'javax.annotation:jsr250-api:1.0'
    compile 'javax.inject:javax.inject:1'
}
```

**참고** : 우리는 gradle에서 제공하는 Android용 annotation processor provided를 사용합니다. dagger-compiler는 빌드하는 동안 dependency graph 클래스를 생성하기 위한 annotation processor repo입니다. 다른 gradle dependency은 Dagger2를 위해 제공됩니다.

### Stpe 2

먼저 데이터 부분을 구축합니다. `User` 클래스 모델을 생성합니다.

```java
public class User {

    private Long id;
    private String name;
    private String address;
    private String createdAt;
    private String updatedAt;

    public User() {
    }

    public User(String name, String address) {
        this.name = name;
        this.address = address;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public String getCreatedAt() {
        return createdAt;
    }

    public void setCreatedAt(String createdAt) {
        this.createdAt = createdAt;
    }

    public String getUpdatedAt() {
        return updatedAt;
    }

    public void setUpdatedAt(String updatedAt) {
        this.updatedAt = updatedAt;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", address='" + address + '\'' +
                ", createdAt='" + createdAt + '\'' +
                ", updatedAt='" + updatedAt + '\'' +
                '}';
    }
}
```

**참조** : 이 클래스는 DB 테이블 데이터를 바인딩합니다.

### Step3

몇 가지 커스텀 어노테이션을 만듭니다.(`ActivityContext`,`AppicationContext`,`DatabaseInfo`,`PerActivity`)

```java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface ActivityContext {
}

@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface ApplicationContext {
}

@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface DatabaseInfo {
}

@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface PerActivity {
}
```

*왜 이러한 주석을 작성하고 `@Qualifier`와 `@Scope`는 무엇입니까?*

`@Qualifier`은 *javax inject 패키지*에 의해 제공되며 의존성을 한정하는데 사용됩니다. 예를들어, 한 클래스는 `ApplicationContext`와 `ActivityContext` 둘 다 요청 할 수 있습니다. 그러나 이 두 객체는 모두 `Context` 유형힙니다. 따라서 Dagger2가 어던 변수에 무엇을 제공해야하는지 파악 하려면 해당 식별자를 명시적으로 지정해야합니다.

따라서 `@Qualifier`는 동일한 유형이지만 다른 인스턴스를 가진 객체를 구변하는데 사용됩니다. 위 코드에서, 우리는 `ApplicationContext`와 `ActivityContext`를 가지므로 주입되는 `Context` 객체가 각각의 `Context` 타입을 참조 할 수 있습니다. `DatabaseInfo`는 클래스 의존성에서 데이터베이스 이름을 제공하는데 사용됩니다. 이후 `String` 클래스가 의존으로 제공되기 때문에 Dagger가 명시적으로 해결 할 수 있도록 항상 `@Qualifier`를 부여하는 것이 좋습니다.

대안으로는 Dagger2가 제공하는 `@Named` 어노테이션입니다. `@Named` 자체는 `@Qualifier`로 어노테이션 처리됩니다. `@Named`를 사용하면 유사 클래스 객체에 문자열 식별자를 제공해야하며 이 식별자는 클래스의 의존성을 매핑하는 데 사용됩니다. 이 예제의 끝에서 `@Named`를 살펴 보겠습니다.

`@Scope`는 의존성 객체가 유지되는 범위를 지정하는데 사용됩니다. 클래스가 의존성을 얻는다면, Scope가 어노테이션된 클래스가 주입된 멤버를 가지며 종속성을 요구하는 해당 클래스의 각 인스턴스는 고유한 멤버 변수 세트를 갖게됩니다.

### Step 4

SQLiteOpenHelper를 상속받는 DbHelper 클래스를 생성합니다. 이 클래스는 모든 DB 관련 작업을 담당합니다.

```java
@Singleton
public class DbHelper extends SQLiteOpenHelper {

    //USER TABLE
    public static final String USER_TABLE_NAME = "users";
    public static final String USER_COLUMN_USER_ID = "id";
    public static final String USER_COLUMN_USER_NAME = "usr_name";
    public static final String USER_COLUMN_USER_ADDRESS = "usr_add";
    public static final String USER_COLUMN_USER_CREATED_AT = "created_at";
    public static final String USER_COLUMN_USER_UPDATED_AT = "updated_at";

    @Inject
    public DbHelper(@ApplicationContext Context context,
                    @DatabaseInfo String dbName,
                    @DatabaseInfo Integer version) {
        super(context, dbName, null, version);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        tableCreateStatements(db);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        db.execSQL("DROP TABLE IF EXISTS " + USER_TABLE_NAME);
        onCreate(db);
    }

    private void tableCreateStatements(SQLiteDatabase db) {
        try {
            db.execSQL(
                    "CREATE TABLE IF NOT EXISTS "
                            + USER_TABLE_NAME + "("
                            + USER_COLUMN_USER_ID + " INTEGER PRIMARY KEY AUTOINCREMENT, "
                            + USER_COLUMN_USER_NAME + " VARCHAR(20), "
                            + USER_COLUMN_USER_ADDRESS + " VARCHAR(50), "
                            + USER_COLUMN_USER_CREATED_AT + " VARCHAR(10) DEFAULT " + getCurrentTimeStamp() + ", "
                            + USER_COLUMN_USER_UPDATED_AT + " VARCHAR(10) DEFAULT " + getCurrentTimeStamp() + ")"
            );

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    protected User getUser(Long userId) throws Resources.NotFoundException, NullPointerException {
        Cursor cursor = null;
        try {
            SQLiteDatabase db = this.getReadableDatabase();
            cursor = db.rawQuery(
                    "SELECT * FROM "
                            + USER_TABLE_NAME
                            + " WHERE "
                            + USER_COLUMN_USER_ID
                            + " = ? ",
                    new String[]{userId + ""});

            if (cursor.getCount() > 0) {
                cursor.moveToFirst();
                User user = new User();
                user.setId(cursor.getLong(cursor.getColumnIndex(USER_COLUMN_USER_ID)));
                user.setName(cursor.getString(cursor.getColumnIndex(USER_COLUMN_USER_NAME)));
                user.setAddress(cursor.getString(cursor.getColumnIndex(USER_COLUMN_USER_ADDRESS)));
                user.setCreatedAt(cursor.getString(cursor.getColumnIndex(USER_COLUMN_USER_CREATED_AT)));
                user.setUpdatedAt(cursor.getString(cursor.getColumnIndex(USER_COLUMN_USER_UPDATED_AT)));
                return user;
            } else {
                throw new Resources.NotFoundException("User with id " + userId + " does not exists");
            }
        } catch (NullPointerException e) {
            e.printStackTrace();
            throw e;
        } finally {
            if (cursor != null)
                cursor.close();
        }
    }

    protected Long insertUser(User user) throws Exception {
        try {
            SQLiteDatabase db = this.getWritableDatabase();
            ContentValues contentValues = new ContentValues();
            contentValues.put(USER_COLUMN_USER_NAME, user.getName());
            contentValues.put(USER_COLUMN_USER_ADDRESS, user.getAddress());
            return db.insert(USER_TABLE_NAME, null, contentValues);
        } catch (Exception e) {
            e.printStackTrace();
            throw e;
        }
    }

    private String getCurrentTimeStamp() {
        return String.valueOf(System.currentTimeMillis() / 1000);
    }
}
```

**참고** : 이 클래스에 사용된 어노테이션을 소개합니다.

1. `@Singleton`은 전역적으로 클래스의 단일 인스턴스를 보장합니다. 따라서 앱의 DbHelper 클래스 인스턴스는 하나뿐이며 클래스가 DbHelper를 의존으로 요청 할 때마다 Dagger의 의존성 그래프에서 유지 관리되는 것은 동일한 인스턴스가 제공됩니다.
2. `@Inject`은 생성자가 클래스 생성 중에 모든 매개 변수 의존성을 모으도록 Dagger에 지시합니다.
3. `@ApplicationContex` 어노테이션을 사용하면 DbHelper가 Dagger의 의존성 그래프에서 앱의 Context 객체를 쉽게 얻을 수 있습니다.
4. `@DatabaseInfo` 어노테이션은 Dagger가 의존성 그래프에서 기존 타입인 문자열과 Integer을 구별하는데 도움이 됩니다.

*우리는 모듈을 다룰 때 다시 이 이야기로 돌아 올 것입니다.*

이 클래스의 나머지 내용은 표준 [`SQLiteOpenHelper`](https://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html)입니다. 이 클래스는 사용자 테이블을 만들고 삽입/읽기를 합니다.

### Step 5
Shared Preferences를 다루는 `SharedPrefsHelper`를 생성합니다.

```java
@Singleton
public class SharedPrefsHelper {

    public static String PREF_KEY_ACCESS_TOKEN = "access-token";

    private SharedPreferences mSharedPreferences;

    @Inject
    public SharedPrefsHelper(SharedPreferences sharedPreferences) {
        mSharedPreferences = sharedPreferences;
    }

    public void put(String key, String value) {
        mSharedPreferences.edit().putString(key, value).apply();
    }

    public void put(String key, int value) {
        mSharedPreferences.edit().putInt(key, value).apply();
    }

    public void put(String key, float value) {
        mSharedPreferences.edit().putFloat(key, value).apply();
    }

    public void put(String key, boolean value) {
        mSharedPreferences.edit().putBoolean(key, value).apply();
    }

    public String get(String key, String defaultValue) {
        return mSharedPreferences.getString(key, defaultValue);
    }

    public Integer get(String key, int defaultValue) {
        return mSharedPreferences.getInt(key, defaultValue);
    }

    public Float get(String key, float defaultValue) {
        return mSharedPreferences.getFloat(key, defaultValue);
    }

    public Boolean get(String key, boolean defaultValue) {
        return mSharedPreferences.getBoolean(key, defaultValue);
    }

    public void deleteSavedData(String key) {
        mSharedPreferences.edit().remove(key).apply();
    }
}
```

**참조** : 이 클래스는 Dagger의 의존성 그래프에서 싱글톤 클래스가 되도록 `@Singleton`으로 어노테이션 처리 되었습니다.

이 클래스는 생성자가 `@Inject`로 표현되었기에 Dagger를 통해 SharedPerferences 의존성을 가져옵니다.

*이 의존성은 어떻게 제공 될 까요? 이 예제의 뒷 부분에서 설명합니다.*

### Step 6

`DataManager` 클래스를 생성합니다.

```java
@Singleton
public class DataManager {

    private Context mContext;
    private DbHelper mDbHelper;
    private SharedPrefsHelper mSharedPrefsHelper;

    @Inject
    public DataManager(@ApplicationContext Context context,
                       DbHelper dbHelper,
                       SharedPrefsHelper sharedPrefsHelper) {
        mContext = context;
        mDbHelper = dbHelper;
        mSharedPrefsHelper = sharedPrefsHelper;
    }

    public void saveAccessToken(String accessToken) {
        mSharedPrefsHelper.put(SharedPrefsHelper.PREF_KEY_ACCESS_TOKEN, accessToken);
    }

    public String getAccessToken(){
        return mSharedPrefsHelper.get(SharedPrefsHelper.PREF_KEY_ACCESS_TOKEN, null);
    }

    public Long createUser(User user) throws Exception {
        return mDbHelper.insertUser(user);
    }

    public User getUser(Long userId) throws Resources.NotFoundException, NullPointerException {
        return mDbHelper.getUser(userId);
    }
}
```

**참조** : 이 클래스는 생성자에서 앱의 Context, DbHelper, SharedPrefsHelper에 대한 의존성을 나타냅니다. 이 의존성들은 앱의 데이터에 접근 할 수 있는 모든 API를 제공합니다.

### Step 7

`android.app.Application`를 상속받는 `DemoApplication`클래스를 생성합니다.

```java
public class DemoApplication extends Application {

    protected ApplicationComponent applicationComponent;

    @Inject
    DataManager dataManager;

    public static DemoApplication get(Context context) {
        return (DemoApplication) context.getApplicationContext();
    }

    @Override
    public void onCreate() {
        super.onCreate();
        applicationComponent = DaggerApplicationComponent
                                    .builder()
                                    .applicationModule(new ApplicationModule(this))
                                    .build();
        applicationComponent.inject(this);
    }

    public ApplicationComponent getComponent(){
        return applicationComponent;
    }
}
```

이 클래스를 `AndroidManifest.xml`에 추가합니다.

```XML
<application
    ...
    android:name=".DemoApplication"
    ...
</application>
```

**참조** : 이 클래스는 DI를 통해 DataManager를 가져옵니다. 이 클래스의 흥미로운 부분은 `ApplicationComponent`입니다. 이 부분을 설명하기 전에 단계를 진행하도록 하겠습니다.

### Step 8

`MainActivity` 클래스를 생성합니다.

```java
public class MainActivity extends AppCompatActivity {

    @Inject
    DataManager mDataManager;

    private ActivityComponent activityComponent;

    private TextView mTvUserInfo;
    private TextView mTvAccessToken;

    public ActivityComponent getActivityComponent() {
        if (activityComponent == null) {
            activityComponent = DaggerActivityComponent.builder()
                    .activityModule(new ActivityModule(this))
                    .applicationComponent(DemoApplication.get(this).getComponent())
                    .build();
        }
        return activityComponent;
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        getActivityComponent().inject(this);

        mTvUserInfo = (TextView) findViewById(R.id.tv_user_info);
        mTvAccessToken = (TextView) findViewById(R.id.tv_access_token);
    }

    @Override
    protected void onPostCreate(@Nullable Bundle savedInstanceState) {
        super.onPostCreate(savedInstanceState);
        createUser();
        getUser();
        mDataManager.saveAccessToken("ASDR12443JFDJF43543J543H3K543");

        String token = mDataManager.getAccessToken();
        if(token != null){
            mTvAccessToken.setText(token);
        }
    }

    private void createUser(){
        try {
            mDataManager.createUser(new User("Ali", "1367, Gurgaon, Haryana, India"));
        } catch (Exception e) {
          e.printStackTrace();
        }
    }

    private void getUser(){
        try {
            User user = mDataManager.getUser(1L);
            mTvUserInfo.setText(user.toString());
        } catch (Exception e) {
          e.printStackTrace();
        }
    }
}
```

또 `activity_main.xml`을 수정합니다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:gravity="center"
    android:orientation="vertical">

    <TextView
        android:id="@+id/tv_user_info"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="@android:color/black"
        android:padding="10dp"/>

    <TextView
        android:id="@+id/tv_access_token"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="@android:color/black"
        android:padding="10dp"/>
</LinearLayout>
```

*이제 약간의 시간을 두고 Dagger2를 살펴보겠습니다.*

1. 클래스에 대한 의존성을 제공하기 위해 `Module` 클래스를 만들어야합니다. 이 클래스는 의존성을 제공하는 메소드를 정의합니다. `Module` 클래스는 `@Module`로 식별되고 의존성 공급 메소드는 `@Provides`로 식별됩니다.
2. `@Inject`을 통한 의존성이 필요한 클래스와 의존성을 제공하는 클래스(`@Module`로 어노테이션 된 클래스) 사이의 연결 역활을 하는 인터페이스를 만들어야합니다.
3. 모듈이 제공해야하는 의존성을 파악하려면 그래프에서 의존성이 필요한 모든 클래스를 스캔 한 다음 제공해야하는 객체의 개수를 계산 해야합니다.

위의 설명을 이해하기 위해 예제 단계로 돌아갑니다.

### Step 9

`DemoApplication` 클래스에 표현된 의존성을 제공해야합니다. 이 클래스에는 `DataManager`가 필요하며 이 클래스를 제공하려면 `DataManager`에 의해 표현된 의존성을 제공해야합니다. 이 의존성은 생성자에서 언급한 `Context`, `DbHelper`, `SharedPrefsHelper`입니다. 그런 다음 그래프에서 더 이동해야합니다.

1. `Context`는 `ApplicationContext`이여야합니다.
2. `DbHelper`에는 `Context`, `dbName` 및 버전이 필요합니다. 이것은 더 이상의 분기가 없습니다.
3. `SharedPrefsHelper`에는 `SharedPreferences`가 필요합니다.

이제 `Context`, `dbName`, `version` 및 `SharedPreferences`와 같은 모든 종속성의 상위 집합을 누적합니다.

이제 이러한 종속성을 제공하기 위해 ApplicationModule을 만듭니다.

```java
@Module
public class ApplicationModule {

    private final Application mApplication;

    public ApplicationModule(Application app) {
        mApplication = app;
    }

    @Provides
    @ApplicationContext
    Context provideContext() {
        return mApplication;
    }

    @Provides
    Application provideApplication() {
        return mApplication;
    }

    @Provides
    @DatabaseInfo
    String provideDatabaseName() {
        return "demo-dagger.db";
    }

    @Provides
    @DatabaseInfo
    Integer provideDatabaseVersion() {
        return 2;
    }

    @Provides
    SharedPreferences provideSharedPrefs() {
        return mApplication.getSharedPreferences("demo-prefs", Context.MODE_PRIVATE);
    }
}
```

**참조** : 이 클래스에는 `@Module` 및 모든 메소드에 `@Provides`를 어노테이션으로 추가했습니다.

이 클래스를 살펴보겠습니다.

1. 생성자에서 `Application` 인스턴스를 전달 받습니다. 이 인스턴스는 다른 의존성을 제공하는데 사용됩니다.
2. 이 클래스는 위의 단계에서 나열한 모든 의존성을 제공합니다.

### Step 10

`DemoApplication` 의존성과 `ApplicationModule`을 연결하는 `Component`를 생성합니다.

```java
@Singleton
@Component(modules = ApplicationModule.class)
public interface ApplicationComponent {

    void inject(DemoApplication demoApplication);

    @ApplicationContext
    Context getContext();

    Application getApplication();

    DataManager getDataManager();

    SharedPrefsHelper getPreferenceHelper();

    DbHelper getDbHelper();

}
```

**참조** : `ApplicationComponent`는 Dagger2에 의해 구현되는 인터페이스입니다. `@Component`를 사용하여 클래스를 `Component`로 지정합니다.

여기서 우리는 `DemoApplication` 인스턴스를 매개변수로 전달하는 `inject()` 메소드를 작성했습니다. *왜 그랬을까요?*

의존성이 필드 주입을 통해, 즉 멤버 변수에 `@Inject`를 통해 의존성이 제공되는 경우 이 인터페이스의 구현을 통해 만들어진 클래스를 검사하도록 Dagger에게 지시해야합니다.

이 클래스는 의존성 그래프에 존재하는 의존성에 엑세스하는데 사용되는 메소드를 제공합니다.

### Step 11

마찬가지로 우리는 `MainActivity`와 그 `Component`를 위한 `Module`을 생성해야합니다. 위의 단계와 동일한 절차를 진행하세요.

```java
@Module
public class ActivityModule {

    private Activity mActivity;

    public ActivityModule(Activity activity) {
        mActivity = activity;
    }

    @Provides
    @ActivityContext
    Context provideContext() {
        return mActivity;
    }

    @Provides
    Activity provideActivity() {
        return mActivity;
    }
}
```

```java
@PerActivity
@Component(dependencies = ApplicationComponent.class, modules = ActivityModule.class)
public interface ActivityComponent {

    void inject(MainActivity mainActivity);

}
```

**참고** : `ActivityComponet`는 `ApplicationComponent` 및 `ActivityModule`을 지정합니다. `ApplicationComponent`는 이전 단계에서 생성된 그래프를 사용하기 위하여 추가되었으며 앱 실행되는 동안 `DemoApplication` 클래스가 유지되기 때문에 `ApplicationComponent` 클래스도 생존합니다.

`@PerActivity`는 `Scope`이며 `Activity`가 생성 될 때마다 `ActivityModule`에서 제공하는 `Context` 및 `Activity`가 인스턴스화 될 것이라고 Dagger에 알리는데 사용됩니다. 따라서 이러한 객체는 해당 `Acitivty`가 살아 있고 각 `Activity`마다 고유한 집합이 있을 때까지 유지 됩니다.

우리는 `DataManager`가 각 `Activity`와 함께 생성 될 것을 요구 할 것입니다. 그러나 `DataManager` 클래스에 `@Singleton`을 어노테이션으로 지정했기 때문에 이는 `Activity`와 함께 생성되지 않습니다. 클래스가 전역 범위에 포함되므로 의존성이 나타나 질 때마다 엑세스 됩니다.

이제 `DemoApplication` 클래스와 `MainActivity` 클래스를 다시 살펴보겠습니다. 이 클래스에는 생성자가 없으며 Android System에서 인스턴스 생성을 담당합니다. 의존성을 얻으려면 `onCreate` 메소드가 각 클래스가 인스턴스화 될 때 한번만 호출이 되므로 `onCreate` 메소드를 이용합니다.

```java
applicationComponent = DaggerApplicationComponent
                            .builder()
                            .applicationModule(new ApplicationModule(this))
                            .build();

applicationComponent.inject(this);
```

`DaggerApplicationComponent`는 Dagger에 의해 생성된 클래스이며 `ApplicationComponent`을 구현합니다. 의존성을 구성하는데 사용된 `ApplicationModule` 클래스를 제공합니다.

우리는 또한 `applicationComponent`의 `inject` 메소드를 호출하고 `DemoApplication`의 인스턴스를 전달합니다. 이는 `DataManager.ApplicationComponent` 인스턴스를 제공하는데 사용하기 위해 수행되며 의존성 그래프에서 사용 가능한 모든 클래스에 엑세스 할 수 있도록 유지되며 엑세스를 위해 명시적으로 사용됩니다.

```java
public ActivityComponent getActivityComponent() {
    if (activityComponent == null) {
        activityComponent = DaggerActivityComponent.builder()
                .activityModule(new ActivityModule(this))
                .applicationComponent(DemoApplication.get(this).getComponent())
                .build();
    }
    return activityComponent;
}
```

`MainActivity`에서도 비슷한 프로세스가 적용됩니다. 유일한 차이점은 `ActivityModule`에서 언급된 의존성 해결에 필요한 `ApplicationComponent`도 제공한다는 것입니다.

**이것으로 Example 프로젝트가 완성되었습니다.**

*Dagger2에 대한 실무 지식을 가졌기를 바랍니다*

우리는 `@Named("string")` 어노테이션에 대해서 언급을 했었는데, `@ApplicationContext` 및 `@ActivityContext`를 `@Named("appication_context")` 및 `@Named("activity_context")`로 대체 할 수 있습니다. 하지만 개인적으로 이 구현이 맘에 들지 않습니다. String TAG를 유지해야 하기 때문입니다.

**참고** : 어떤 이유 때문에 우리가 안드로이드 생성 클래스를 필드 주입을 통해 의존성을 주입해야한다면 그 클래스의 컴포너틑 인터페이스를 정의하고 클래스의 생성자에서 주입 메소드를 호출하십시오. 이러한 방식을 수행하지 않고, 주입을 위해 `@Inject`를 사용하여 생성자를 사용하려고 시도하는 방법을 알아내는 것이 좋습니다.
