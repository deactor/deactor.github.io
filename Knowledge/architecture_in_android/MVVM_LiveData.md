---
title:  "MVVM LiveData与LifeCycle笔记"
---

# LiveData与LifeCycle相关总结
## LifeCycle
>Lifecycle 是一个类，用于存储有关组件（如 Activity 或 Fragment）的生命周期状态的信息，并允许其他对象观察此状态。  
>Lifecycle 使用Event和State跟踪其关联组件的生命周期状态

官方Activity生命周期的状态和事件图：
![LifeCycle](https://raw.githubusercontent.com/deactor/deactor.github.io/master/imgs/mvvm_lifecycle.png)

使用：
1. 实现LifecycleOwner的对象为被观察者
2. 实现LifecycleObserver的对象为观察者。
3. LifecycleOwner.addObserver(LifecycleObserver),被观察者添加观察者。则被观察者状态变化就会通知到观察者。

## ViewModel
> 此处的 ViewModel是指lifecycle包下的ViewModel类。该类负责为界面准备数据。

ViewModel 对象存在的时间范围是获取 ViewModel 时传递给 ViewModelProvider 的 Lifecycle。ViewModel 将一直留在内存中，直到对应的 Lifecycle 永久消失：对于 Activity，是在 Activity finish（注意不是onDestory）时；而对于 Fragment，是在 Fragment 分离时。

>**注意**：ViewModel 绝不能引用视图、Lifecycle 或可能存储对 Activity 上下文的引用的任何类。

### ViewModel的创建
官方todo-demo中的创建方式：
```java
//TasksActivity.java中
public static TasksViewModel obtainViewModel(FragmentActivity activity) {
    // 继承ViewModelProvider.NewInstanceFactory，重写了创建对象的create方法。
    ViewModelFactory factory = ViewModelFactory.getInstance(activity.getApplication());

    TasksViewModel viewModel =
            ViewModelProviders.of(activity, factory).get(TasksViewModel.class);

    return viewModel;
}
```
官方文档中的创建方式：
```java
public class MyActivity extends AppCompatActivity {
    public void onCreate(Bundle savedInstanceState) {
        // Create a ViewModel the first time the system calls an activity's onCreate() method.
        // Re-created activities receive the same MyViewModel instance created by the first activity.

        MyViewModel model = new ViewModelProvider(this).get(MyViewModel.class);
    }
}
```
>ViewModelProviders.of(activity, factory)内部最终也是去new ViewModelProvider，不过相比直接new的方式，进行了合法性检测，如果viewModel是在activity的onCreate之前创建则抛出异常。

#### 两种方式的具体创建过程：
1. ViewModelProviders.of()方式
    ```java
    public static ViewModelProvider of(@NonNull FragmentActivity activity,
                                        @Nullable Factory factory) {
        // 检查合法性，确定activity已attach到application，即防止viewMode在activity的onCreate之前调用。
        Application application = checkApplication(activity);
        if (factory == null) {
            // 没有传递factory，就使用ViewModelProvider中的AndroidViewModelFactory
            factory = ViewModelProvider.AndroidViewModelFactory.getInstance(application);
        }
        // 仍然是new ViewModelProvider
        return new ViewModelProvider(activity.getViewModelStore(), factory);
    }
    ```
2. new ViewModelProvider()方式
    ```java
    // lifecycle-viewmodel2.0包下ViewModelProvider的两个构造方法：
    // 最终就是设置mFactory变量和mViewModelStore。
    public ViewModelProvider(@NonNull ViewModelStoreOwner owner, @NonNull Factory factory) {
        this(owner.getViewModelStore(), factory);
    }

    public ViewModelProvider(@NonNull ViewModelStore store, @NonNull Factory factory) {
        mFactory = factory;
        this.mViewModelStore = store;
    }
    ```
>ViewModelStore是什么？  
ViewModelStore内部是一个hashMap，对外提供put，get，clear方法。  
此处ViewModelProvider中持有的ViewModelStore是来自于activity的。FragmentActivity实现了ViewModelStoreOwner接口，可以提供ViewModelStore。

#### ViewModelProvider.get()返回ViewModle的过程：
```java
public <T extends ViewModel> T get(@NonNull Class<T> modelClass) {
    String canonicalName = modelClass.getCanonicalName();
    if (canonicalName == null) {
        throw new IllegalArgumentException("Local and anonymous classes can not be ViewModels");
    }
    return get(DEFAULT_KEY + ":" + canonicalName, modelClass);
}

// 先按照key，在mViewModelStore中查找，没有则使用mFactory创建并添加到mViewModelStore中。
public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
    ViewModel viewModel = mViewModelStore.get(key);

    if (modelClass.isInstance(viewModel)) {
        //noinspection unchecked
        return (T) viewModel;
    } else {
        //noinspection StatementWithEmptyBody
        if (viewModel != null) {
            // TODO: log a warning.
        }
    }

    viewModel = mFactory.create(modelClass);
    mViewModelStore.put(key, viewModel);
    //noinspection unchecked
    return (T) viewModel;
}
```
ViewModelProvider中提供两个Factory，一个NewInstanceFactory，一个AndroidViewModelFactory。
+ NewInstanceFactory：反射调用modelClass的默认构造方法创建modelClass。
+ AndroidViewModelFactory：对于AndroidViewModel.class，反射调用带application的构造参数，非AndroidViewModel.class调用默认构造参数进行对象创建。

#### 什么时候自定义Factory？
需要调用ViewModel带参构造函数时，如官方todo-demo中的ViewModelFactory。其create方法中是调用对应ViewModel的带参构造函数，将Repository传给ViewModel。
```java
@Override
public <T extends ViewModel> T create(Class<T> modelClass) {
    if (modelClass.isAssignableFrom(StatisticsViewModel.class)) {
        //noinspection unchecked
        return (T) new StatisticsViewModel(mTasksRepository);
    } else if (modelClass.isAssignableFrom(TaskDetailViewModel.class)) {
        //noinspection unchecked
        return (T) new TaskDetailViewModel(mTasksRepository);
    } else if (modelClass.isAssignableFrom(AddEditTaskViewModel.class)) {
        //noinspection unchecked
        return (T) new AddEditTaskViewModel(mTasksRepository);
    } else if (modelClass.isAssignableFrom(TasksViewModel.class)) {
        //noinspection unchecked
        return (T) new TasksViewModel(mTasksRepository);
    }
    throw new IllegalArgumentException("Unknown ViewModel class: " + modelClass.getName());
}
```

#### 为什么说Activity中ViewModel的生命周期是在Activity finish时？
因为ViewModle创建后会存放到Activity的ViewModelStore中，这个ViewModelStore就是在非config变化导致的onDestory时被clear的。
```java
protected void onDestroy() {
    super.onDestroy();

    if (mViewModelStore != null && !isChangingConfigurations()) {
        mViewModelStore.clear();
    }

    mFragments.dispatchDestroy();
}
```


## LiveData应用
### LiveData与DataBinding的关系：
没有LiveData之前，DataBinding中使用ObservableField来描述可观察的数据，而ObservableField是不能感知LifeCycle的。LiveData实现了LifeCycle的感知，可以说在DataBinding中LiveData就是用来替代ObservableField的。xml中的使用方式不变，java中LiveData不再需要@Bindable.

### LiveData添加观察者：
通过DataBinding，xml中view已经实现了对LiveData的观察，对LiveData的变化做出反应。  
我们仍然可以对LiveData手动添加观察者，实现事件驱动。以官方todo-demo为例：
>TasksActivity中，从ViewModle获取OpenTaskEvent LiveData对象，向其添加匿名Observer，当OpenTaskEvent值变化时，该匿名Observer的onChanged方法即会触发。
```java
// TasksActivity.java
public class TasksActivity extends AppCompatActivity implements TaskItemNavigator, TasksNavigator {
    private TasksViewModel mViewModel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // ...

        mViewModel = obtainViewModel(this);

        // Subscribe to "open task" event
        mViewModel.getOpenTaskEvent().observe(this, new Observer<Event<String>>() {
            @Override
            public void onChanged(Event<String> taskIdEvent) {
                String taskId = taskIdEvent.getContentIfNotHandled();
                if (taskId != null) {
                    openTaskDetails(taskId);
                }

            }
        });
    }

    // ...

    @Override
    public void openTaskDetails(String taskId) {
        Intent intent = new Intent(this, TaskDetailActivity.class);
        intent.putExtra(TaskDetailActivity.EXTRA_TASK_ID, taskId);
        startActivityForResult(intent, AddEditTaskActivity.REQUEST_CODE);

    }
}
```
```java
// TasksViewModel.java
public class TasksViewModel extends ViewModel {
    private final MutableLiveData<Event<String>> mOpenTaskEvent = new MutableLiveData<>();

    public LiveData<Event<String>> getOpenTaskEvent() {
        return mOpenTaskEvent;
    }

    void openTask(String taskId) {
        mOpenTaskEvent.setValue(new Event<>(taskId));

    }
}
```

### LiveData的observe方法
observe的两个参数，LifecycleOwner是观察者，Observer应该叫观察者观察到数据变化后的回调。LiveData感知到的Lifecycle是LifecycleOwner的。
```java
public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer) {
    assertMainThread("observe"); //检查是否在主线程，不在则抛异常
    if (owner.getLifecycle().getCurrentState() == DESTROYED) {//检查Lifecycle当前状态
        // ignore
        return;
    }
    // LiveData的内部类，根据lifeCycle的状态判断是否分发change事件给observer,及在lifeCycle的Destory状态时移除observer，observer还是应该叫callback更合适。
    LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);

    // mObservers是一个SafeIterableMap，以observer为key，wrapper为value。
    // 当liveData执行set的时候，会遍历mObservers，取出wrapper，根据owner的状态，执行observer的onChange方法。
    ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
    if (existing != null && !existing.isAttachedTo(owner)) {
        throw new IllegalArgumentException("Cannot add the same observer"
                + " with different lifecycles");
    }
    if (existing != null) {
        return;
    }

    // 为什么要把wrapper给owner？？？？因为wrapper要观察owner的lifecyle。
    // 上面的代码保证在livedata值变化时，通知给owner（set时遍历mObservers）。
    // 下面这行代码保证当owner由非活跃状态变为活跃状态时能立即收到livedata（即使livedata没变化）。wrapper会接收owner的onStateChange，从而根据状态将data给owner。
    owner.getLifecycle().addObserver(wrapper);
}
```
>关于owner由非活跃状态变为活跃状态时能立即收到livedata：  
+ owner第一次由非活跃状态变为活跃状态时，会立即收到livedata的值。
+ owner非第一次由非活跃状态变为活跃状态时，仅当非活跃状态时，livedata有变化，才会收到livedata的最新值。

### LifeCycle的状态以及LiveData怎么判断LifyCycle是否活跃
>ComponentActivity.java实现了LifecycleOwner接口成为一个LifecycleOwner，可提供lifecycle，其提供的lifycyle为Lifecycle子类LifecycleRegistry（内部对外提供状态设置和切换接口）。

>LiveData判断活跃：内部子类ObserverWrapper存储active状态，根据此变量判断是否为活跃状态，同时通过activeStateChanged来修改此变量。ObserverWrapper子类LifecycleBoundObserver在state至少为start时会设置active（即start和resume状态）为true。其他状态为false。在removeObserver时也会设置为false。

```java
// ComponentActivity.java中
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // 注入一个ReportFragment，通过该对象，从framework对activity的lifecycle state进行管理
    ReportFragment.injectIfNeededIn(this);
}

@Override
protected void onSaveInstanceState(Bundle outState) {
    mLifecycleRegistry.markState(Lifecycle.State.CREATED);
    super.onSaveInstanceState(outState);
}
```
LifeCycle中的事件,在activity生命周期函数别调用时，就会发出这些事件（由ReportFrgment代为发出）。
```java
// 事件
public enum Event {
    ON_CREATE, // Constant for onCreate event of the {@link LifecycleOwner}.
    ON_START, // Constant for onStart event of the {@link LifecycleOwner}.
    ON_RESUME, // Constant for onResume event of the {@link LifecycleOwner}.
    ON_PAUSE, // Constant for onPause event of the {@link LifecycleOwner}.
    ON_STOP, // Constant for onStop event of the {@link LifecycleOwner}.
    ON_DESTROY, // Constant for onDestroy event of the {@link LifecycleOwner}.
    ON_ANY // An {@link Event Event} constant that can be used to match all events.
}
// 状态
public enum State {
    DESTROYED, // 对于activity，会Activity#onDestroy()调用之前处于该状态。
    INITIALIZED, // 对于activity，会在Activity已构造但还未调用onCreate时处于该状态。
    CREATED, // 对于activity，在activity.onCreate之后或onStop之前会处于该状态。
    STARTED, // 对于activity，在activity.onStart之后或onPause之前会处于该状态。
    RESUMED; // 对于activity，在activity.onResume被调用之后处于该状态。
}

// 根据事件，状态的转换方式
static State getStateAfter(Event event) {
    switch (event) {
        case ON_CREATE:
        case ON_STOP:
            return CREATED;
        case ON_START:
        case ON_PAUSE:
            return STARTED;
        case ON_RESUME:
            return RESUMED;
        case ON_DESTROY:
            return DESTROYED;
        case ON_ANY:
            break;
    }
    throw new IllegalArgumentException("Unexpected event value " + event);
}
```

```java
public abstract class LiveData<T> {
    // ...

    // 移除observer时将对于observer标记为非活跃状态。
    public void removeObserver(@NonNull final Observer<? super T> observer) {
        assertMainThread("removeObserver");
        ObserverWrapper removed = mObservers.remove(observer);
        if (removed == null) {
            return;
        }
        removed.detachObserver();
        removed.activeStateChanged(false);
    }

    class LifecycleBoundObserver extends ObserverWrapper implements GenericLifecycleObserver {
        @NonNull
        final LifecycleOwner mOwner;

        // 判断至少为STARTED状态为active状态。
        @Override
        boolean shouldBeActive() {
            return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);
        }

        @Override
        public void onStateChanged(LifecycleOwner source, Lifecycle.Event event) {
            if (mOwner.getLifecycle().getCurrentState() == DESTROYED) {
                removeObserver(mObserver);
                return;
            }
            activeStateChanged(shouldBeActive());
        }
    }
}
```

### LiveData及其子类
+ LiveData：内部数据不可变，未提供public的set方法。
+ MutableLiveData：提供setValue和postValue方法更新LiveData数据，即可变的LiveData。
  + setValue：需要在主线程中执行。
  + postValue：可以在子线程中执行。
+ MediatorLiveData:合并多个 LiveData 源。只要任何原始的 LiveData 源对象发生更改，就会触发 MediatorLiveData 对象的观察者。同样未提供public的set方法。


>ViewModel 应只会向观察者公开不可变的 LiveData 对象，即不应该让观察者来修改LiveData。如官方Todo-demo：
` private final MutableLiveData<Event<String>> mOpenTaskEvent = new MutableLiveData<>();`对外的getter方法`public LiveData<Event<String>> getOpenTaskEvent() {return mOpenTaskEvent;}`返回的是LiveData而不是MutableLiveData。

## 问题
### ViewModel中不能存context，碰到需要context的处理怎么办？
通过事件驱动，放到能拥有context的地方进行处理。


