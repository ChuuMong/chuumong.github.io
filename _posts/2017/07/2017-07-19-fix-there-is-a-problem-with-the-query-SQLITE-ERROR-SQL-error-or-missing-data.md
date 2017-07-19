---
layout: post
category : Android
title : "[Bug Fix] There is a problem with the query: [SQLITE_ERROR] SQL error or missing database 에러 해결"
description : ""
tags : [Android, Bug Fix]
---

{% include JB/setup %}

# There is a problem with the query: [SQLITE_ERROR] SQL error or missing database 에러 해결

얼마전 [Android Architecture Components](https://developer.android.com/topic/libraries/architecture/index.html)로 샘플 소스를 구현 중 `There is a problem with the query: [SQLITE_ERROR] SQL error or missing database` 에러가 발생

빌드 중 UserDao에서 발생하는 에러

Room 관련 소스는 아래와 같이 작성

```kotlin
Room.databaseBuilder(app, UserDatabase::class.java, "user").build()
```

```kotlin
@Database(entities = arrayOf(UserEntity::class), version = 1)
abstract class UserDatabase : RoomDatabase() {
    abstract fun getUserDao(): UserDao
}
```

```kotlin
@Dao
interface UserDao {
...

	@Query("SELECT * FROM user")
	fun getAll(): LiveData<List<UserEntity>>

...
}
```

```kotlin
@Entity
data class UserEntity
```

Query 질의 시 table을 user로 설정, 실제로 생성된 table 이름과 다르기 때문에 발생한 에러

`UserDao_Impl.java`를 보면 `getAll()`에 `_result`가 선언되어있지 않은 상태로 return되고 있음

실제 `Room.databaseBuilder`에서 table이 생성되는 줄 알았는데 아니였고 `Entiry`클래스 이름으로 생성

그렇기 때문에 다른 이름의 table을 사용하려면 

```kotlin
@Entity(tableName = "user")
data class UserEntity
```

로 사용

아래는 [Room 공식 문서](https://developer.android.com/topic/libraries/architecture/room.html)를 일부 발췌

> By default, Room uses the class name as the database table name. If you want the table to have a different name, set the tableName property of the @Entity annotation, as shown in the following code snippet:

> @Entity(tableName = "users")
class User {
    ...
}

> Caution: Table names in SQLite are case insensitive.
