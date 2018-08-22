# LiveData

## Reference

https://developer.android.com/reference/android/arch/lifecycle/LiveData

## 장점

- 기존에는 데이터 내용 변경 시 데이터가 어느 시점에 업데이트 되었음을 알지 못하는데 LiveData를 사용하면 변경 내용을 적용할 수 있음
- LifecycleOwner 를 통해 변경된 Lifecycle에 대해 핸들링할 필요가 없이 데이터 유지


## 사용방법

**(!) 최근에 올린 데이터베이스 업데이트에서 지원되니 업데이트 이후 사용**

**LiveData 지원 함수**

- LocationDao.loadAllWithLive()
- SceneDao
    - loadByRowIdWithLive(rowId)
    - loadByIdWithLive(sceneId)
    - loadAllAreCapturedWithLive()

```java
AppDatabase db = AppDatabase.getInstance(...);

db.locationDao.loadAllWithLive().observe(
    /** LifecycleOwner (Activity, Fragment) **/ this,
    /** Observer Interface **/ locations -> {
    
    // locations 데이터 처리 메인스레드이므로 runOnMain 사용 X
});
```