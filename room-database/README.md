# Room Database

## 사용방법

1. RoomDatabase를 상속한 AppDatabase를 가져온다.
```java
AppDatabase.getInstance(Context context)

// Activity
AppDatabase db = AppDatabase.getInstance(this);

// Fragment
AppDatabase db = AppDatabase.getInstance(getActivity());
// or
AppDatabase db = AppDatabase.getInstance(getContext());
```

2. AppDatabase에서 Dao 객체를 가져온다.
```java
MediaDao dao = db.mediaDao();
```

3. Dao에 정의된 함수를 호출하여 사용하되, DB 작업은 메인 스레드에서 처리할 수 없으므로 정의된 AppExecutors를 사용하여 다른 스레드에서 호출한다.
```java
AppDatabase db = AppDatabase.getInstance(getActivity());
AppExecutors executors = new AppExecutors();
executors.diskIO().execute(() -> {
    Media media = ...;
    db.mediaDao().insert(media);
    executors.mainThread().execute(() -> {
        // 메인 스레드에서 데이터 처리
        showMedia(media);
    });
});
```