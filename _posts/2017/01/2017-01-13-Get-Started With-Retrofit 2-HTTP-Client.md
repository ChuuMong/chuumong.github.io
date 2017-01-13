---
layout: post
category : Android
title : "[Retrofit] Retrofit 2 HTTP 클라이언트 시작하기"
description : ""
tags : [Android, Retrofit]
---

{% include JB/setup %}

> *이 문서는 [Get Started With Retrofit 2 HTTP Client](https://code.tutsplus.com/tutorials/getting-started-with-retrofit-2--cms-27792)를 번역한 문서입니다.*

# Retrofit 2 HTTP 클라이언트 시작하기

![What You'll Be Createing](https://cms-assets.tutsplus.com/uploads/users/1499/posts/27792/final_image/gt5.JPG)

## Retrofit?

[Retrofit](https://square.github.io/retrofit/)은 Android 및 Java용 Type-Safe(예측 불가능한 결과를 내지 않는) HTTP 클라이언트입니다. Retrofit을 사용하면 API를 Java 인터페이스로 변환하여 RECT 웹 서비스에 쉽게 연결 할 수 있습니다. 이 튜토리얼에서는 aNDROID에서 가장 많이 사용되고 자주 추천되는 HTTP 중 하나를 사용하는 방법에 대해서 설명합니다.

이 강력한 라이브러리를 사용하면 JSON 또는 XML 데이터를 쉽게 처리 할 수 있으며 이 데이터는 [POJO](https://ko.wikipedia.org/wiki/Plain_Old_Java_Object)(단순한 자바 오브젝트)로 파싱 됩니다. `GET`, `POST`, `PUT`, `PATCH`, 그리고 `DELETE` Request를 모두 실행 될 수 있습니다.

대부분의 오픈 소스 소프트웨어와 마찬가지로 Retrofit은 다른 강력한 라이브러리 및 Tool 위에 구현되어있습니다. Retrofit은 네트워크 Request를 처리하기 위해서 [OKHttp](http://square.github.io/okhttp/)를 사용합니다. 또한 Retrofit에는 JSON에서 Java 객체로 변환 해 줄 JSON 변환기가 내장되어있지 않기 때문에 아래와 같은 JSON 변환기 라이브러리에 대한 지원을 제공합니다.

- Gson: `com.squareup.retrofit2:converter-gson`
- Jackson: `com.squareup.retrofit2:converter-jackson`
- Moshi: `com.squareup.retrofit2:converter-moshi`

[Protocol Buffers](https://developers.google.com/protocol-buffers/)의 경우 Retrofit은 다음을 지원합니다.

- Protobuf: `com.squareup.retrofit2:converter-protobuf`
- Wire: `com.squareup.retrofit2:converter-wire`

그리고 XML에 대해서 Retrofit은 다음을 지원합니다.

- Simple Framework: `com.squareup.retrofit2:converter-simpleframework`

## 왜 Retrofit을 사용해야 할까?

REST API와의 인터페이스를 위한 자신만의 Type-Safe HTTP 라이브러리를 개발하는것은 정말 고통스러울 수 있습니다. Connection 만들기, 캐싱, 실패한 요청 다시 시도, 스레딩, Response 파싱, 에러 핸들링등과 같은 많은 기능을 처리해야합니다. 반면, Retrofit은 매우 좋은 설계가 되어있고 문서화, 테스트가 되어있어 귀중한 시간과 고통을 줄여주는 검증 된 라이브러리입니다.

이 튜토리얼에서는 Retrofit2를 사용하여 [Stack Exchange](https://api.stackexchange.com/docs) API 최근 답변에 대해서 쿼리를 처리하는 네트워크 Request 간단한 앱에 대해서 설명 할 것입니다. 우리는 베이스 URL https://api.stackexchange.com/2.2/에 엔드 포인트(`\answers`) 추가하여 결과를 가져오는 `GET` Request 수행 한 뒤 결과를 [RecyclerView](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html)에 표현합니다. 또한 상태 또는 데이터의 흐름을 쉽게 관리하기 위해서 [RxJava](https://github.com/ReactiveX/RxJava)를 이용하여 작업을 수행하는 방법에 대해서 설명합니다.

### 1. Android Studio 프로젝트 생성

Android Studio를 실행하고 MainActivity라는 Empty Activity로 새 프로젝트를 만듭니다.

![](https://cms-assets.tutsplus.com/uploads/users/1499/posts/27792/image/a2.png)


### 2. Dependencie 선언

새 프로젝트를 생성한 다음 `build.gradle`에 아래의 Dependencie를 선언하세요. Dependencie에는 RecyclerView, Retrofit, Retrofit에 통합된 JSON을 POJO로 변환하는 Google의 Gson 라이브러리들이 포함됩니다.

```
// Retrofit
compile 'com.squareup.retrofit2:retrofit:2.1.0'

// JSON Parsing
compile 'com.google.code.gson:gson:2.6.1'
compile 'com.squareup.retrofit2:converter-gson:2.1.0'

// recyclerview
compile 'com.android.support:recyclerview-v7:25.0.1'
```

라이브러리를 추가하기 위해서는 프로젝트를 동기화 해야하는 것을 잊지 마십시오.

### 3. Internet 권한 추가

네트워크 작업을 수행하기 위해서는 어플리케이션 매니패스트(AndroidManifast.xml)에 INTERNET 권한을 추가해야합니다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.chikeandroid.retrofittutorial">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
            android:allowBackup="true"
            android:icon="@mipmap/ic_launcher"
            android:label="@string/app_name"
            android:supportsRtl="true"
            android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>

                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>

</manifest>
```

### 4. 모델 자동 생성

매우 유용한 도구 인 [jsonschema2pojo](http://www.jsonschema2pojo.org/)를 활용하여 JSON을 자동으로 모델을 만들려고합니다.

#### 샘플 JSON 데이터 가져 오기

브라우저의 주소 표시줄에 https://api.stackexchange.com/2.2/answers?order=desc&sort=activity&site=stackoverflow를 복사하여 붙여 넣거나 [Postman](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en)에 익숙한 경우 해당 툴을 사용합니다. 그런 다음 Enter 키를 누릅니다. 그러면 주어진 엔드 포인트(URL)에서 GET Reqeust가 실행됩니다. Response로 볼 수 있는것은 JSON의 배열입니다. 아래의 스크린 샷 은 Postman을 사용한 JSON Response입니다.

![](https://cms-assets.tutsplus.com/uploads/users/769/posts/27792/image/1.jpg)

```
{
  "items": [
    {
      "owner": {
        "reputation": 1,
        "user_id": 6540831,
        "user_type": "registered",
        "profile_image": "https://www.gravatar.com/avatar/6a468ce8a8ff42c17923a6009ab77723?s=128&d=identicon&r=PG&f=1",
        "display_name": "bobolafrite",
        "link": "http://stackoverflow.com/users/6540831/bobolafrite"
      },
      "is_accepted": false,
      "score": 0,
      "last_activity_date": 1480862271,
      "creation_date": 1480862271,
      "answer_id": 40959732,
      "question_id": 35931342
    },
    {
      "owner": {
        "reputation": 629,
        "user_id": 3054722,
        "user_type": "registered",
        "profile_image": "https://www.gravatar.com/avatar/0cf65651ae9a3ba2858ef0d0a7dbf900?s=128&d=identicon&r=PG&f=1",
        "display_name": "jeremy-denis",
        "link": "http://stackoverflow.com/users/3054722/jeremy-denis"
      },
      "is_accepted": false,
      "score": 0,
      "last_activity_date": 1480862260,
      "creation_date": 1480862260,
      "answer_id": 40959731,
      "question_id": 40959661
    },
    ...
  ],
  "has_more": true,
  "backoff": 10,
  "quota_max": 300,
  "quota_remaining": 241
}
```

JSON Response를 브라우저 또는 Postman에서 복사하세요.

#### JSON 데이터를 Java에 매핑

이제 jsonschema2pojo에 접속하여 JSON Response를 붙여넣으세요.

**Source Type**을 JSON으로 선택하고 **Annotation style**을 Gson으로 선택 한 뒤 **Allow additional properties**의 선택을 해제하세요.

![](https://cms-assets.tutsplus.com/uploads/users/1499/posts/27792/image/u99.jpg)

그런 다음 **Preview** 버튼을 클릭하여 Java 객체를 생성하세요.

![](https://cms-assets.tutsplus.com/uploads/users/769/posts/27792/image/kpo09.jpg)

생성 된 소스를 보면 `@SerializedName` 및 `@Expose` 어노테이션이 있는데 이 중 `@SerializedName`는 Gson이 JSON 키를 필드에 매핑하기 위해서 필요합니다. 클래스 멤버 변수 이름에 대한 Java의 [camelCase](https://en.wikipedia.org/wiki/Camel_case)의 규칙을 유지하기 위해서 변수의 단어를 구분 할 때 밑줄을 사용하지 않는 것이 좋습니다. `@SerializedName`은 JSON 키와 멤버 변수 이름 사이의 변환을 돕습니다.

```java
@SerializedName("quota_remaining")
@Expose
private Integer quotaRemaining;
```

위 예제에서 JSON 키 `quota_remaining`가 Java 필드 `quotaRemaining`에 매핑되어야 한다고 Gson에 알립니다. JSON 키가 Java 필드와 마찬가지로 `quotaRemaining`인 경우 Gson이 자동으로 `@SerializedName` 어노테이션이 필요하지 않습니다.(하지만 어플리케이션을 Release 할 때 소스코드가 난독화 되어지기 때문에 Java 필드가 변환됩니다. 그렇기 때문에  `@SerializedName`는 필수로 사용하는 것이 좋습니다.)

`@Expose` 어노테이션은 이 필드가 JSON 직렬화 또는 비 직렬화에 노출되어야 함을 나타냅니다.

#### 데이터 모델을 Android Studio로 가져 오기

이제 안드로이드 스튜디오로 돌아옵니다. 메인 패키지 안에 **data** 패키지를 생성합니다. 새로이 생성된 data 패키지 안에 **model** 패키지를 생성합니다. model 패키지 안에 `Owner` 클래스를 생성합니다. 이제 jsonschema2pojo에서 생성한 POJO를 복사하여 Owner 클래스에 붙여 넣습니다.

```java
import com.google.gson.annotations.Expose;
import com.google.gson.annotations.SerializedName;

public class Owner {

    @SerializedName("reputation")
    @Expose
    private Integer reputation;
    @SerializedName("user_id")
    @Expose
    private Integer userId;
    @SerializedName("user_type")
    @Expose
    private String userType;
    @SerializedName("profile_image")
    @Expose
    private String profileImage;
    @SerializedName("display_name")
    @Expose
    private String displayName;
    @SerializedName("link")
    @Expose
    private String link;
    @SerializedName("accept_rate")
    @Expose
    private Integer acceptRate;
}
```
jsonschema2pojo에서 복사 한 Item 클래스도 동일한 작업을 수행합니다.

```java
import com.google.gson.annotations.Expose;
import com.google.gson.annotations.SerializedName;

public class Item {

    @SerializedName("owner")
    @Expose
    private Owner owner;
    @SerializedName("is_accepted")
    @Expose
    private Boolean isAccepted;
    @SerializedName("score")
    @Expose
    private Integer score;
    @SerializedName("last_activity_date")
    @Expose
    private Integer lastActivityDate;
    @SerializedName("creation_date")
    @Expose
    private Integer creationDate;
    @SerializedName("answer_id")
    @Expose
    private Integer answerId;
    @SerializedName("question_id")
    @Expose
    private Integer questionId;
    @SerializedName("last_edit_date")
    @Expose
    private Integer lastEditDate;
}
```

마지막으로 StackOverflow 답변을 리턴 받을 수 있는 `SOAnswersResponse` 클래스를 생성합니다. jsonschema2pojo에서의 이 클래스의 이름은 `Example`입니다. 이름을 `SOAnswersResponse`으로 변경하였는지 확인하세요.

```java
import com.google.gson.annotations.Expose;
import com.google.gson.annotations.SerializedName;

import java.util.List;

public class SOAnswersResponse {

    @SerializedName("items")
    @Expose
    private List<Item> items = null;
    @SerializedName("has_more")
    @Expose
    private Boolean hasMore;
    @SerializedName("backoff")
    @Expose
    private Integer backoff;
    @SerializedName("quota_max")
    @Expose
    private Integer quotaMax;
    @SerializedName("quota_remaining")
    @Expose
    private Integer quotaRemaining;
}
```

### 5. Retrofit 인스턴스 만들기

Retrofit을 사용하여 REST API에 네트워크 Request를 보내려면 `Retrofit.Builder` 클래스를 사용하여 인스턴스를 만들고 Base URL을 구성해야합니다.

data 패키지 안에 **remote**라는 서브 패키지를 생성합니다. 이제 remote 패키지 내부에 RetrofitClient라는 이름의 Java 클래스를 생성합니다. 이 클래스는 Retrofit의 [싱글 톤]()을 생성합니다. Retrofit에는 인스터스를 빌드하기 위한 Base URL이 필요하므로 `RetrofitClient.getClient(String baseUrl)`을 호출 할 때 URL을 전달합니다. 이 URL은 13행에 인스턴스를 빌드하는데 사용됩니다. 또한 14번째 줄에는 JSON 변환기(Gson)을 지정하고 있습니다.

```java
import retrofit2.Retrofit;
import retrofit2.converter.gson.GsonConverterFactory;

public class RetrofitClient {

    private static Retrofit retrofit = null;

    public static Retrofit getClient(String baseUrl) {
        if (retrofit==null) {
            retrofit = new Retrofit.Builder()
                    .baseUrl(baseUrl)
                    .addConverterFactory(GsonConverterFactory.create())
                    .build();
        }
        return retrofit;
    }
}
```

### 6. API Interface 만들기

remote 패키지 안에, `SOService`라는 이름의 Interface를 생성합니다. 이 Interface는 `GET`, `POST`, `PUT`, `PATCH`, `DELETE`와 같은 HTTP Request를 처리하는데 사용 할 메소드를 포함하고 있습니다. 이 튜토리얼에서는 `GET` Reqeust을 사용합니다.

```java
import com.chikeandroid.retrofittutorial.data.model.SOAnswersResponse;

import java.util.List;

import retrofit2.Call;
import retrofit2.http.GET;

public interface SOService {

   @GET("/answers?order=desc&sort=activity&site=stackoverflow")
   Call<SOAnswersResponse> getAnswers();

   @GET("/answers?order=desc&sort=activity&site=stackoverflow")
   Call<SOAnswersResponse> getAnswers(@Query("tagged") String tags);
}
```

`@GET` 어노테이션은 해당 메소드가 호출 되면 실행된 `GET` Request를 명시적으로 정의합니다. 이 인터페이스의 모든 메소드에는 Request 메소드와 상대 URL을 지정하는 HTTP 어노테이션이 있어야합니다. 사용 할 수 있는 내장 어노테이션은 `@GET`, `@POST`, `@PUT`, `@DELETE`, `@HEAD` 입니다.

두 번째 메서드 정의에서는 서버의 데이터를 필터링하기 위해 쿼리 필드를 추가했습니다. Retrofit에는 엔드 포인트에서 하드 코딩하는 대신 `@Query("key")` 어노테이션을 사용 할 수 있습니다. 키 값은 URL의 필드를 나타냅니다. Retrofit을 통해서 URL에 추가됩니다. 예를 들어, `getAnswers(String tags)` 메소드에 `"android"`값을 인수로 전달하면 전체 URL은 다음과 같습니다.


`https://api.stackexchange.com/2.2/answers?order=desc&sort=activity&site=stackoverflow&tagged=android`

인터페이스 메소드의 매개변수는 다음과 같은 어노테이션을 가질 수 있습니다.

| 어노테이션 | 설명 |
|-|-|
|`@Path`| API 엔드포인트의 변수를 대채|
|`@Query`|어노테이션의 매개변수 값으로 쿼리 키 이름을 지정|
|`@Body` |Post 호출의 페이로드|
|`@Header`|어노테이션의 매개변수 값으로 헤더를 지정|

### 7. API 유틸 만들기

이제 유틸리티 클래스를 생성합니다. `ApiUtlis`라는 이름입니다. 이 클래스는 statis 변수인 base URL을 가지고 있고 `getSOService()` static 메소드를 통해 SOService 인터페이스를 어플리케이션에게 제공합니다.

```java
public class ApiUtils {

    public static final String BASE_URL = "https://api.stackexchange.com/2.2/";

    public static SOService getSOService() {
        return RetrofitClient.getClient(BASE_URL).create(SOService.class);
    }
}
```

### 8. RecyclerView에 출력하기

API 호출에 대한 결과를 [RecyclerView](https://code.tutsplus.com/tutorials/getting-started-with-recyclerview-and-cardview-on-android--cms-23465)에 표현하기 위해서는 adapter가 필요합니다. 다음 코드는 `AnswersAdapter` 클래스의 일부를 보여줍니다.

```java
public class AnswersAdapter extends RecyclerView.Adapter<AnswersAdapter.ViewHolder> {

    private List<Item> mItems;
    private Context mContext;
    private PostItemListener mItemListener;

    public class ViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener{

        public TextView titleTv;
        PostItemListener mItemListener;

        public ViewHolder(View itemView, PostItemListener postItemListener) {
            super(itemView);
            titleTv = (TextView) itemView.findViewById(android.R.id.text1);

            this.mItemListener = postItemListener;
            itemView.setOnClickListener(this);
        }

        @Override
        public void onClick(View view) {
            Item item = getItem(getAdapterPosition());
            this.mItemListener.onPostClick(item.getAnswerId());

            notifyDataSetChanged();
        }
    }

    public AnswersAdapter(Context context, List<Item> posts, PostItemListener itemListener) {
        mItems = posts;
        mContext = context;
        mItemListener = itemListener;
    }

    @Override
    public AnswersAdapter.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {

        Context context = parent.getContext();
        LayoutInflater inflater = LayoutInflater.from(context);

        View postView = inflater.inflate(android.R.layout.simple_list_item_1, parent, false);

        ViewHolder viewHolder = new ViewHolder(postView, this.mItemListener);
        return viewHolder;
    }

    @Override
    public void onBindViewHolder(AnswersAdapter.ViewHolder holder, int position) {

        Item item = mItems.get(position);
        TextView textView = holder.titleTv;
        textView.setText(item.getOwner().getDisplayName());
    }

    @Override
    public int getItemCount() {
        return mItems.size();
    }

    public void updateAnswers(List<Item> items) {
        mItems = items;
        notifyDataSetChanged();
    }

    private Item getItem(int adapterPosition) {
        return mItems.get(adapterPosition);
    }

    public interface PostItemListener {
        void onPostClick(long id);
    }
}
```

### 9. Request 수행하기

`MainActivity`의 `onCreate()` 메소드 안에 `SOService` 인터페이스의 인스턴스, RecyclerView, adapter를 초기화 합니다. 마지막에 `loadAnswers()` 메소드를 호출합니다.

```java
   private AnswersAdapter adapter;
   private RecyclerView recyclerView;
   private SOService service;

   @Override
   protected void onCreate (Bundle savedInstanceState)  {
       super.onCreate( savedInstanceState );
       setContentView(R.layout.activity_main );
       service = ApiUtils.getSOService();
       recyclerView = (RecyclerView) findViewById(R.id.rv_answers);
       adapter = new AnswersAdapter(this, new ArrayList<Item>(0), new AnswersAdapter.PostItemListener() {

           @Override
           public void onPostClick(long id) {
               Toast.makeText(MainActivity.this, "Post id is" + id, Toast.LENGTH_SHORT).show();
           }
       });

       RecyclerView.LayoutManager layoutManager = new LinearLayoutManager(this);
       recyclerView.setLayoutManager(layoutManager);
       recyclerView.setAdapter(adapter);
       recyclerView.setHasFixedSize(true);
       RecyclerView.ItemDecoration itemDecoration = new DividerItemDecoration(this, DividerItemDecoration.VERTICAL_LIST);
       recyclerView.addItemDecoration(itemDecoration);

       loadAnswers();
   }
```

`loadAnswers()` 메소드는 `enqueue()`를 호출하여 네트워크 Request를 처리합니다. Response가 들어오면 Retrofit은 JSON Response에 대한 Java 객체로 파싱합니다.(이것은 GsonConverter를 사용하여 가능합니다)

```java
public void loadAnswers() {
    service.getAnswers().enqueue(new Callback<SOAnswersResponse>() {
        @Override
        public void onResponse(Call<SOAnswersResponse> call, Response<SOAnswersResponse> response) {
            if(response.isSuccessful()) {
                mAdapter.updateAnswers(response.body().getItems());
                Log.d("MainActivity", "posts loaded from API");
            }else {
                int statusCode  = response.code();
                // handle request errors depending on status code
            }
        }

        @Override
        public void onFailure(Call<SOAnswersResponse> call, Throwable t) {
           showErrorMessage();
            Log.d("MainActivity", "error loading from API");

        }
    });
}
```

### 10. `enqueue()`의 이해

`enqueue()`는 비동기로 Request를 보내고 Response가 돌아 왔을 때 콜백으로 앱에게 알립니다. 이 Request는 비동기식이므로 Retrofit에서 Main UI 스레드가 차단되거나 간섭받지 않도록 Background 스레드에서 Request를 처리합니다.

`enqueue()`를 사용하려면 2개의 콜백 메소드를 구현해야합니다.

- `onResponse()`
- `onFailure()`

이 메소드 중 하나만 주어진 Request에 대해 Response 전달받아 처리합니다.

- `onResponse()` : 수신된 HTTP Response에 대해 호출됩니다. 이 메소드는 서버가 오류 메시지를 리턴하는 경우에도 올바르게 처리 할 수 있는 Response를 위해 호출됩니다. 따라서 상태 코드가 404 또는 500 일 때도 이 메소드가 호출됩니다. 상태 코드를 처리하기 위해서 상태 코드를 얻으려면 `response.code()`을 사용하세요. `isSuccessful()` 메소드를 사용하여 상태 코드가 성공을 나타내는 200~300 범위에 있는지도 확인 할 수 있습니다.

- `onFailure()` : 서버와 통신하는 중 네트워크 예외가 발생하거나 Request를 처리하거나 Reponse를 처리하는 중에 예기치 않은 예외가 발생되었을 때 호출됩니다.

동기 Reuqest를 수행하려면 `execute()` 메소드를 사용하세요. 메인/UI 스레드의 동기 메소드는 모든 사용자 작업을 차단합니다. 따라서 Android의 메인/UI 스레드에서 동기 메소드를 실행하지 마세요! 대신, Background 스레드에서 실행하세요.

### 11. App 테스팅

이제 앱을 실행하여 확인해보세요.

![](https://cms-assets.tutsplus.com/uploads/users/1499/posts/27792/image/gt5.JPG)

### 12. RxJava 적용

RxJava 팬이라면 RxJava로 Retrofit을 쉽게 구현 할 수 있습니다. Retrofit1에서는 기본적으로 통합되었지만 Retrofit2에서는 몇 가지 추가 dependencie를 포함시켜야합니다. Retrofit은 Call 인스턴스를 실행하기 위한 기본 어댑터와 함께 제공됩니다. 따라서 RxJava CallAdapter를 포함하여 RxJava를 포함하도록 Retrofit의 실행 메커니즘을 변경 할 수 있습니다.

#### Step 1

dependencie 추가

```
compile 'io.reactivex:rxjava:1.1.6'
compile 'io.reactivex:rxandroid:1.2.1'
compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'
```

#### Step 2

Rtrofit 인스턴스를 빌드 할 때 새로운 CallAdapter `RxJavaCallAdapterFactory.create()`를 추가하세요.

```java
public static Retrofit getClient(String baseUrl) {
    if (retrofit==null) {
        retrofit = new Retrofit.Builder()
                .baseUrl(baseUrl)
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .addConverterFactory(GsonConverterFactory.create())
                .build();
    }
    return retrofit;
}
```

#### Step 3

Request를 수행 할 때, subscriber는 `SOAnswersResponse`에서 이벤트를 발생시키는 observable 스트림에 응답합니다. `onNext` 메소드는 subscriber가 방출 된 이벤트를 수신 한 다음 어댑터로 전달 할 때 호출됩니다.

```java
@Override
public void loadAnswers() {
    mService.getAnswers().subscribeOn(Schedulers.io()).observeOn(AndroidSchedulers.mainThread())
            .subscribe(new Subscriber<SOAnswersResponse>() {
                @Override
                public void onCompleted() {

                }

                @Override
                public void onError(Throwable e) {

                }

                @Override
                public void onNext(SOAnswersResponse soAnswersResponse) {
                    mAdapter.updateAnswers(soAnswersResponse.getItems());
                }
            });
}
```

Ashraff Hathibelagal의 [Getting Started With ReactiveX on Android](https://code.tutsplus.com/tutorials/getting-started-with-reactivex-on-android--cms-24387)를 확인하면 RxJava 및 RxAndroid에 대해 자세히 알아볼 수 있습니다.
