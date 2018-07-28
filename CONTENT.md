# RecyclerView

## RecyclerView 장점
- ViewHolder 패턴을 통해 View를 재사용하여 많은 아이템을 넣더라도 성능이 저하되지 않음
- 아이템 추가, 삭제와 같은 변경사항이 있을 때 애니메이션을 쉽게 적용할 수 있음

## RecyclerView 구조
![](images/recyclerview_overview.png)

## Gradle에 추가
```gradle
// AndroidX를 사용하지 않을 경우
implementation 'com.android.support:recyclerview-v7:27.1.1' 

// AndroidX를 사용할 경우
implementation "androidx.recyclerview:recyclerview:1.0.0-beta01"
```
## XML에 추가
- 예시
```xml
<!-- activity_todo.xml -->
<android.support.v7.widget.RecyclerView
    android:id="@+id/recycler_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>

<!-- 각각의 Item을 구현할 때 사용하는 레이아웃 파일 -->
<!-- item_todo.xml -->
<LinearLayout ...>
  <TextView android:id="@+id/tv_title" ... />
  <TextView android:id="@+id/tv_desc" ... />
</LinearLayout>
```

## ViewHolder
- 정의된 내용
```java
// ViewHolder는 아래와 같이 정의되어 있으며, 생성시에 View를 제공해야한다
public abstract static class ViewHolder {
  public ViewHolder(@NonNull View itemView) {
    ...
  }
}
```

- 사용 예시
```java
public class ToDoViewHolder extends RecyclerView.ViewHolder {
  TextView titleView;
  TextView descView;

  public ToDoViewHolder(View itemView) {
    super(itemView);
    // 제공받은 View로부터 각각의 View들을 findViewById 로 찾아 변수로 저장한다
    // View의 id 값은 item_todo.xml 에 정의된 View 들의 Id를 사용하면 된다
    titleView = itemView.findViewById(R.id.tv_title);
    descView = itemView.findViewById(R.id.tv_desc);
  }
}
```

## Adapter
- 정의된 내용
```java
// RecyclerView.Adapter 는 아래와 같이 정의되어 있으며,
// Adapter 구현 시에 위에서 RecyclerView.ViewHolder 를 상속 받아 만든
// ViewHolder를 제너릭 자리에 추가해야한다
public class RecyclerView extends ViewGroup implements ScrollingView, NestedScrollingChild2 {
  ...
  public abstract static class Adapter<VH extends ViewHolder> {
    ...
  }

  // 아래의 세 메소드는 무조건 구현을 해야한다
  // View를 아래에서 생성하여 ViewHolder를 생성하는 메소드
  @NonNull
  public abstract VH onCreateViewHolder(@NonNull ViewGroup parent, int viewType);

  // ViewHolder의 View들에 데이터를 바인딩하는 메소드
  public abstract void onBindViewHolder(@NonNull VH holder, int position);

  // RecyclerView에게 ItemSet의 크기가 얼마인지 알려주는 메소드
  public abstract int getItemCount();
}
```

- 사용 예시
``` java
public class ToDoAdapter extends RecyclerView.Adapter<ToDoViewHolder> {
  // ToDo 객체는 구현되어있다고 가정하며,
  // 이 변수는 생성자나 함수를 통해 값을 부여하면 된다
  // 반드시 리스트일 필요는 없으며 배열이나,
  // 혹은 객체를 사용하여 표현할 수 있다
  List<ToDo> todos;

  // 함수를 통한 ItemSet 지정
  // 새로 지정
  public void setTodos(List<ToDo> newTodos) {
    this.todos.clear();
    this.todos.addAll(newTodos);
    // 이 메소드를 사용하여 아이템이 변경되었음을 RecyclerView에게 알려야하며,
    // 사용해야만 리스트 화면이 업데이트된다.
    notifyDataSetChanged();
  }

  // 추가
  public void addTodo(ToDo todo) {
    todos.add(todo);
    // 이 메소드를 사용하여 아이템 추가 시에 기본 애니메이션인 Fade-in 효과를 적용할 수 있으며,
    // 해당 메소드나 notifyDataSetChanged() 를 사용하여 RecyclerView에게 아이템이 변경되었음
    // 을 알려야 한다. 알리면 리스트 화면이 업데이트된다.
    notifyItemInserted(todos.size());
  }

  public void deleteTodo(int position) {
    todos.remove(position);
    // 이 메소드를 사용하여 아이템 삭제 시에 기본 애니메이션인 Fade-out 효과를 적용할 수 있으며,
    // 해당 메소드나 notifyDataSetChanged() 를 사용하여 RecyclerView에게 아이템이 변경되었음
    // 을 알려야 한다. 알리면 리스트 화면이 업데이트된다.
    notifyItemRemoved(position);
  }

  @Override
  public ToDoViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
    Context context = parent.getContext();
    LayoutInflater inflater = LayoutInflater.from(context);
    // View를 생성할 때 각 아이템에 사용할 레이아웃 파일이 지정되어 있어야한다.
    // 이 예시에서는 위에서 생성한 item_todo.xml 을 사용한다.
    View view = inflater.inflate(R.layout.item_todo, parent, false);
    return new ToDoViewHolder(view);
  }

  @Override
  public void onBindViewHolder(ToDoViewHolder holder, int position) {
    // position 값을 사용하여 아이템의 위치에 맞게 값을 View에 적용하면 된다
    // position은 0 ~ getItemCount() - 1 의 값이 사용된다
    // e.g position이 0일때, 0번 ToDo를 가져온다
    ToDo todo = todos.get(position);
    holder.titleView.setText(todo.title);
    holder.descView.setText(todo.desc);
  }

  @Override
  public int getItemCount() {
    return todos.size();
  }
}
```

## Activity (or Fragment) 에서 사용

```java
// 이전 안드로이드 개발 시에는 findViewById를 사용할 때 저장하는 변수에 맞게
// 캐스팅해야했지만 targetSdkVersion 이 26 이상이면 생략할 수 있다
// 
// targetSdkVersion 확인 : (앱 모듈)/build.gradle > android.defaultConfig 내 포함
List<ToDo> todos = ...;
ToDoAdapter adapter = new ToDoAdapter(todos); // 생성자에 itemSet 추가
adapter.addTodo(new Todo(...)); // 함수를 통한 Item 추가

RecyclerView recyclerView = findViewById(R.id.recycler_view);
recyclerView.setLayoutManager(new LinearLayoutManager(getContext()));
// Item 들로 인해 RecyclerView 사이즈가 변경되어야한다면 false 로 하며,
// 변경되지 않고 고정된 크기를 가진다면 성능향상을 위해 true로 지정한다
recyclerView.setHasFixedSize(true);
// 생성한 adapter 를 저장한다.
recyclerView.setAdapter(mAdapter);
```

## 추가

- RecyclerView를 사용하게 될 경우 아이템을 터치하였을때 터치 피드백이 없는데 이는 item 레이아웃에 아래의 background 속성과 clickable 속성을 추가하면 피드백을 확인할 수 있다
```xml
<LinearLayout 
  android:background="?android:attr/selectableItemBackground"
  android:clickable="true"
  ...>
</LinearLayout>
```

- RecyclerView를 사용할 경우 각 아이템에 대한 터치 이벤트를 기본 어댑터로는 확인할 수 없는데, 터치 이벤트를 아래와 같이 직접 구현하면 사용할 수 있다. (전체를 터치하여 처리할 경우)

> 예시 1 (Adapter 내부에서 처리)

```java
public class ToDoAdapter extends RecyclerView.Adapter<ToDoViewHolder> {
  ...
  @Override
  public void onBindViewHolder(ToDoViewHolder holder, int position) {
    ...
    holder.itemView.setOnClickListener(view -> {
      // 아이템 터치 시의 행동을 정의
    });
  }
```
> 예시 2 (Adapter 외부에서 처리)

```java
interface OnItemClickListener {

  void onItemClicked(Todo todo);

}

public class ToDoAdapter extends RecyclerView.Adapter<ToDoViewHolder> {

  OnItemClickListener listener;

  public ToDoAdapter(OnItemClickListener listener) {
    this.listener = listener;
  }

  ...

  @Override
  public void onBindViewHolder(ToDoViewHolder holder, int position) {
    final ToDo todo = ...
    ...
    holder.itemView.setOnClickListener(view -> {
      if (this.listener != null) {
        listener.onItemClicked(todo);
      }
    });
  }
}

// Adapter를 생성할때
ToDoAdapter adapter = new ToDoAdapter(todo -> {
  // 아이템 터치 시의 행동을 정의
});
```