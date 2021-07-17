---
title:  "MVVM DataBinding笔记"
---

### DataBinding
#### 数据绑定类：
系统会为每个布局文件生成一个绑定类。默认情况下，类名称基于布局文件的名称，它会转换为驼峰大小写形式并在末尾添加 Binding 后缀。如：
>activity_main.xml -> ActivityMainBinding 需要xml中按照databinding的方式添加layout顶层标签，不然不会生成binding类,data标签不是生成binding类的必要条件，layout一定要。  

文件路径(Gradle6.5直接在AS中搜索，检索不到对应文件)：
+ ActivityMainBinding:
    + Gradle4.1：build/intermediates/classes/../databinding/ActivityMainBinding.class
    + Gradle6.5: build/generated/data_binding_base_class_source_out/../databinding/ActivityMainBinding.java  
>Gradle6.5（其他版本可能也会）下，会在build/generated/ap_generated_sources/../databinding下生成ActivityMainBindingImpl文件并继承ActivityMainBinding。

在Activity中可以使用如下两种方式获取Binding对象：Binding对象可以访问到xml中带id的view组件，即不再需要findViewById。
```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // 方式一，inflate仅扩充布局，仍然需要setContentView才能显示。内部实际调用的DataBindingUtil.inflate
        ActivityMainBinding binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());

        // 方式二，使用DataBindingUtil，直接就可以显示。
        ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);

        // DataBindingUtil.inflate和DataBindingUtil.setContentView最终都是走到DataBindingUtil.bind方法，
        //只不过DataBindingUtil.setContentView会通过传入的activity对象执行activity.setContentView(R.layout.activity_main)
        // 个人觉得还是方式二比较方便直观。
    }
}
```

#### 数据更新时序图 （待添加）

#### 事件处理
+ 方法引用：利用view自身的onClick等属性设置触发方法。就是普通的绑定方法的方式，需要方法签名必须与监听器对象中的方法（如OnClickListener的onclick方法）签名完全一致。`android:onClick="@{viewModel::onButtonClicked}"`
+ 监听器绑定：也是利用view的onclick等属性，但是赋的是表达式，可以传递额外的参数，不需要签名和onClick方法一致。`android:onClick="@{() -> viewModel.onButtonClicked(view.visible)}"`
+ 也可以还按照以前的方式，在代码里`setOnClickListener(new onClickListener{...})`;

> 方法引用与监听器绑定区别：  
>1. 方法引用和监听器绑定之间的主要区别在于实际监听器实现是在绑定数据时创建的，而不是在事件触发时创建的。如果您希望在事件发生时对表达式求值，则应使用监听器绑定。监听器绑定是在事件发生时运行的绑定表达式。它们类似于方法引用，但允许您运行任意数据绑定表达式。
>2. 在方法引用中，方法的参数必须与事件监听器的参数匹配。在监听器绑定中，只有您的返回值必须与监听器的预期返回值相匹配（预期返回值无效除外）。

### 问题：
1. bingding类提供了infalte和bind两个方法，有什么区别？  
    + inflate内部是调用DataBindingUtil.inflate，该方法会先使用inflate来扩充布局，然后再调用bind方法，返回binding对象。  
    + bind内部是调用DataBindingUtil.bind，该方法直接返回binding对象（从DataBinderMapper中查找返回ActivityMainBindingImpl）。
    + 以上，所以二者的区别主要就是是否进行inflate。
2. BR文件是什么？
    + 数据绑定库在模块包中生成一个名为BR的类，其中包含用于数据绑定的资源的ID（fieldId），路径同绑定类。每个@Bindable修饰的属性都会在该文件中有个ID。

#### @Bindable在程序中的使用：
+ 修饰属性，该属性不需要是observable数据类型，BR中会生成对应的fieldId，xml中可以引用，但数据变化需要手动进行notifyPropertyChanged(BR.属性名)。
+ 修饰get方法，如getName，即使没有声明name属性，系统也会在BR中生成对应的fieldId（name），xml中可以引用，同样的需要手动通知变化。

> Observable的属性，其set方法内部已经添加了notifyChange方法来通知属性变化。所以不需要再手动notifyChange。

### 绑定类生成的代码：
#### xml中不添加data标签时：
``` xml
<!-- activity_main.xml 内容如下： -->
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <android.support.constraint.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Hello World!"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

    </android.support.constraint.ConstraintLayout>
</layout>
```

``` java
/** 
 *  提供了两个方法，bind和inflate，最终都是调用DataBindingUtil的bind和inflate方法。
 *  这两个方法中均已经将layoutId设置为了R.layout.activity_main，该类就是根据该xml生成的，自然知道此id。
 *  所以当在activity中调用inflate来扩充布局时不需要传递layoutId。
 */
public abstract class ActivityMainBinding extends ViewDataBinding {
  protected ActivityMainBinding(Object _bindingComponent, View _root, int _localFieldCount) {
    super(_bindingComponent, _root, _localFieldCount);
  }

  @NonNull
  public static ActivityMainBinding inflate(@NonNull LayoutInflater inflater,
      @Nullable ViewGroup root, boolean attachToRoot) {
    return inflate(inflater, root, attachToRoot, DataBindingUtil.getDefaultComponent());
  }

  @NonNull
  @Deprecated
  public static ActivityMainBinding inflate(@NonNull LayoutInflater inflater,
      @Nullable ViewGroup root, boolean attachToRoot, @Nullable Object component) {
    return ViewDataBinding.<ActivityMainBinding>inflateInternal(inflater, R.layout.activity_main, root, attachToRoot, component);
  }

  @NonNull
  public static ActivityMainBinding inflate(@NonNull LayoutInflater inflater) {
    return inflate(inflater, DataBindingUtil.getDefaultComponent());
  }

  @NonNull
  @Deprecated
  public static ActivityMainBinding inflate(@NonNull LayoutInflater inflater,
      @Nullable Object component) {
    return ViewDataBinding.<ActivityMainBinding>inflateInternal(inflater, R.layout.activity_main, null, false, component);
  }

  public static ActivityMainBinding bind(@NonNull View view) {
    return bind(view, DataBindingUtil.getDefaultComponent());
  }

  @Deprecated
  public static ActivityMainBinding bind(@NonNull View view, @Nullable Object component) {
    return (ActivityMainBinding)bind(component, view, R.layout.activity_main);
  }
```

``` java
// 生成的binding类的实现类内容ActivityMainBindingImpl.java，看起来是处理数据绑定逻辑的。
public class ActivityMainBindingImpl extends ActivityMainBinding  {

    @Nullable
    private static final android.databinding.ViewDataBinding.IncludedLayouts sIncludes;
    @Nullable
    private static final android.util.SparseIntArray sViewsWithIds;
    static {
        sIncludes = null;
        sViewsWithIds = null;
    }
    // views
    @NonNull
    private final android.support.constraint.ConstraintLayout mboundView0;
    // variables
    // values
    // listeners
    // Inverse Binding Event Handlers

    public ActivityMainBindingImpl(@Nullable android.databinding.DataBindingComponent bindingComponent, @NonNull View root) {
        this(bindingComponent, root, mapBindings(bindingComponent, root, 1, sIncludes, sViewsWithIds));
    }
    private ActivityMainBindingImpl(android.databinding.DataBindingComponent bindingComponent, View root, Object[] bindings) {
        super(bindingComponent, root, 0
            );
        this.mboundView0 = (android.support.constraint.ConstraintLayout) bindings[0];
        this.mboundView0.setTag(null);
        setRootTag(root);
        // listeners
        invalidateAll();
    }

    @Override
    public void invalidateAll() {
        synchronized(this) {
                mDirtyFlags = 0x1L;
        }
        requestRebind();
    }

    @Override
    public boolean hasPendingBindings() {
        synchronized(this) {
            if (mDirtyFlags != 0) {
                return true;
            }
        }
        return false;
    }

    @Override
    public boolean setVariable(int variableId, @Nullable Object variable)  {
        boolean variableSet = true;
            return variableSet;
    }

    @Override
    protected boolean onFieldChange(int localFieldId, Object object, int fieldId) {
        switch (localFieldId) {
        }
        return false;
    }

    @Override
    protected void executeBindings() {
        long dirtyFlags = 0;
        synchronized(this) {
            dirtyFlags = mDirtyFlags;
            mDirtyFlags = 0;
        }
        // batch finished
    }
    // Listener Stub Implementations
    // callback impls
    // dirty flag
    private  long mDirtyFlags = 0xffffffffffffffffL;
    /* flag mapping
        flag 0 (0x1L): null
    flag mapping end*/
    //end
}
```
#### xml中添加data标签，数据非Observable时：
修改activity_main.xml，添加data部分后，绑定类的内容如何变化。
```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">
    <data>
        <variable
            name="view"
            type="com.qdd.mvvmtest.MainActivity" />
    </data>

    <android.support.constraint.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Hello World!"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

    </android.support.constraint.ConstraintLayout>
</layout>
```
```java
// 增加了对应data的属性和setter getter方法，其他内容没变
public abstract class ActivityMainBinding extends ViewDataBinding {
  // 增加了一个Bindable注解修饰的mView变量。
  @Bindable
  protected MainActivity mView;

  protected ActivityMainBinding(Object _bindingComponent, View _root, int _localFieldCount) {
    super(_bindingComponent, _root, _localFieldCount);
  }

  // 生成set 抽象方法
  public abstract void setView(@Nullable MainActivity view);

  // 生成get方法
  @Nullable
  public MainActivity getView() {
    return mView;
  }

  @NonNull
  public static ActivityMainBinding inflate(@NonNull LayoutInflater inflater,
      @Nullable ViewGroup root, boolean attachToRoot) {
    return inflate(inflater, root, attachToRoot, DataBindingUtil.getDefaultComponent());
  }

  @NonNull
  @Deprecated
  public static ActivityMainBinding inflate(@NonNull LayoutInflater inflater,
      @Nullable ViewGroup root, boolean attachToRoot, @Nullable Object component) {
    return ViewDataBinding.<ActivityMainBinding>inflateInternal(inflater, R.layout.activity_main, root, attachToRoot, component);
  }

  @NonNull
  public static ActivityMainBinding inflate(@NonNull LayoutInflater inflater) {
    return inflate(inflater, DataBindingUtil.getDefaultComponent());
  }

  @NonNull
  @Deprecated
  public static ActivityMainBinding inflate(@NonNull LayoutInflater inflater,
      @Nullable Object component) {
    return ViewDataBinding.<ActivityMainBinding>inflateInternal(inflater, R.layout.activity_main, null, false, component);
  }

  public static ActivityMainBinding bind(@NonNull View view) {
    return bind(view, DataBindingUtil.getDefaultComponent());
  }

  @Deprecated
  public static ActivityMainBinding bind(@NonNull View view, @Nullable Object component) {
    return (ActivityMainBinding)bind(component, view, R.layout.activity_main);
  }
}
```
```java
public class ActivityMainBindingImpl extends ActivityMainBinding  {

    @Nullable
    private static final android.databinding.ViewDataBinding.IncludedLayouts sIncludes;
    @Nullable
    private static final android.util.SparseIntArray sViewsWithIds;
    static {
        sIncludes = null;
        sViewsWithIds = null;
    }
    // views
    @NonNull
    private final android.support.constraint.ConstraintLayout mboundView0;
    // variables
    // values
    // listeners
    // Inverse Binding Event Handlers

    public ActivityMainBindingImpl(@Nullable android.databinding.DataBindingComponent bindingComponent, @NonNull View root) {
        this(bindingComponent, root, mapBindings(bindingComponent, root, 1, sIncludes, sViewsWithIds));
    }
    private ActivityMainBindingImpl(android.databinding.DataBindingComponent bindingComponent, View root, Object[] bindings) {
        super(bindingComponent, root, 0
            );
        this.mboundView0 = (android.support.constraint.ConstraintLayout) bindings[0];
        this.mboundView0.setTag(null);
        setRootTag(root);
        // listeners
        invalidateAll();
    }

    @Override
    public void invalidateAll() {
        synchronized(this) {
                mDirtyFlags = 0x2L; // 从0x1L变成了0x2L
        }
        requestRebind();
    }

    @Override
    public boolean hasPendingBindings() {
        synchronized(this) {
            if (mDirtyFlags != 0) {
                return true;
            }
        }
        return false;
    }

    @Override
    public boolean setVariable(int variableId, @Nullable Object variable)  {
        boolean variableSet = true;
        // 添加了具体data的set
        if (BR.view == variableId) {
            setView((com.qdd.mvvmtest.MainActivity) variable);
        }
        else {
            variableSet = false;
        }
            return variableSet;
    }

    // 实现了data的set方法
    public void setView(@Nullable com.qdd.mvvmtest.MainActivity View) {
        this.mView = View;
    }

    @Override
    protected boolean onFieldChange(int localFieldId, Object object, int fieldId) {
        switch (localFieldId) {
        }
        return false;
    }

    @Override
    protected void executeBindings() {
        long dirtyFlags = 0;
        synchronized(this) {
            dirtyFlags = mDirtyFlags;
            mDirtyFlags = 0;
        }
        // batch finished
    }
    // Listener Stub Implementations
    // callback impls
    // dirty flag
    private  long mDirtyFlags = 0xffffffffffffffffL;
    /* flag mapping
        flag 0 (0x1L): view
        flag 1 (0x2L): null
    flag mapping end*/
    //end
}
```
#### xml中添加data标签，数据为Observable，xml中未进行调用时：
```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">
    <data>
        <variable
            name="view"
            type="com.qdd.mvvmtest.MainActivity" />
        <variable
            name="viewModel"
            type="com.qdd.mvvmtest.MainViewModel" />
    </data>

    <android.support.constraint.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Hello World!"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

    </android.support.constraint.ConstraintLayout>
</layout>
```
```java
public class MainViewModel extends BaseObservable {
    public final ObservableField<String> textStr = new ObservableField<>();
}
```
```java
public abstract class ActivityMainBinding extends ViewDataBinding {
  @Bindable
  protected MainActivity mView;

  @Bindable
  protected MainViewModel mViewModel;

  protected ActivityMainBinding(Object _bindingComponent, View _root, int _localFieldCount) {
    super(_bindingComponent, _root, _localFieldCount);
  }

  public abstract void setView(@Nullable MainActivity view);

  @Nullable
  public MainActivity getView() {
    return mView;
  }

  public abstract void setViewModel(@Nullable MainViewModel viewModel);

  @Nullable
  public MainViewModel getViewModel() {
    return mViewModel;
  }

    // ... 相比较之前，增加了mViewModel属性及getter setter方法。
}
```
```java
public class ActivityMainBindingImpl extends ActivityMainBinding  {

    // ... 与之前相同的代码已省略

    @Override
    public void invalidateAll() {
        synchronized(this) {
                mDirtyFlags = 0x4L; // 从0x2L变成了0x4L
        }
        requestRebind();
    }

    @Override
    public boolean setVariable(int variableId, @Nullable Object variable)  {
        boolean variableSet = true;
        if (BR.view == variableId) {
            setView((com.qdd.mvvmtest.MainActivity) variable);
        }
        else if (BR.viewModel == variableId) { // 增加了ViewMode的设置
            setViewModel((com.qdd.mvvmtest.MainViewModel) variable);
        }
        else {
            variableSet = false;
        }
            return variableSet;
    }

    // 增加了ViewMode的setter实现方法
    public void setViewModel(@Nullable com.qdd.mvvmtest.MainViewModel ViewModel) {
        this.mViewModel = ViewModel;
    }

    @Override
    protected boolean onFieldChange(int localFieldId, Object object, int fieldId) {
        switch (localFieldId) {
            case 0 : // 增加了onChangeViewModel的分支处理
                return onChangeViewModel((com.qdd.mvvmtest.MainViewModel) object, fieldId);
        }
        return false;
    }
    // 增加了onChangeViewModel方法
    private boolean onChangeViewModel(com.qdd.mvvmtest.MainViewModel ViewModel, int fieldId) {
        if (fieldId == BR._all) {
            synchronized(this) {
                    mDirtyFlags |= 0x1L;
            }
            return true;
        }
        return false;
    }
    // ...
}
```
#### xml中添加data标签，数据为Observable，xml中进行调用时：
```xml
<!-- text改为引用 viewModel.textStr -->
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{viewModel.textStr}"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toRightOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
```
```java
public abstract class ActivityMainBinding extends ViewDataBinding {
  // ... 和上个文件一样，没有变化
}
```
```java
public class ActivityMainBindingImpl extends ActivityMainBinding  {

    @Nullable
    private static final android.databinding.ViewDataBinding.IncludedLayouts sIncludes;
    @Nullable
    private static final android.util.SparseIntArray sViewsWithIds;
    static {
        sIncludes = null;
        sViewsWithIds = null;
    }
    // views
    @NonNull
    private final android.support.constraint.ConstraintLayout mboundView0;
    @NonNull
    private final android.widget.TextView mboundView1; // 增加了对应的boundView.
    // variables
    // values
    // listeners
    // Inverse Binding Event Handlers

    public ActivityMainBindingImpl(@Nullable android.databinding.DataBindingComponent bindingComponent, @NonNull View root) {
        this(bindingComponent, root, mapBindings(bindingComponent, root, 2, sIncludes, sViewsWithIds));
    }
    private ActivityMainBindingImpl(android.databinding.DataBindingComponent bindingComponent, View root, Object[] bindings) {
        super(bindingComponent, root, 2
            );
        this.mboundView0 = (android.support.constraint.ConstraintLayout) bindings[0];
        this.mboundView0.setTag(null);
        this.mboundView1 = (android.widget.TextView) bindings[1]; // 设置新增的boundView
        this.mboundView1.setTag(null);
        setRootTag(root);
        // listeners
        invalidateAll();
    }

    @Override
    public void invalidateAll() {
        synchronized(this) {
                mDirtyFlags = 0x8L; // 从0x4L变为0x8L，all的标志位，表明所有数据都要更新
        }
        requestRebind();
    }

    @Override
    public boolean hasPendingBindings() {
        synchronized(this) {
            if (mDirtyFlags != 0) {
                return true;
            }
        }
        return false;
    }

    @Override
    public boolean setVariable(int variableId, @Nullable Object variable)  {
        boolean variableSet = true;
        if (BR.view == variableId) {
            setView((com.qdd.mvvmtest.MainActivity) variable);
        }
        else if (BR.viewModel == variableId) {
            setViewModel((com.qdd.mvvmtest.MainViewModel) variable);
        }
        else {
            variableSet = false;
        }
            return variableSet;
    }

    public void setView(@Nullable com.qdd.mvvmtest.MainActivity View) {
        this.mView = View;
    }
    public void setViewModel(@Nullable com.qdd.mvvmtest.MainViewModel ViewModel) {
        updateRegistration(1, ViewModel);
        this.mViewModel = ViewModel;
        synchronized(this) {
            mDirtyFlags |= 0x2L;
        }
        notifyPropertyChanged(BR.viewModel);
        super.requestRebind();
    }

    @Override
    protected boolean onFieldChange(int localFieldId, Object object, int fieldId) {
        switch (localFieldId) {
            case 0 : // 增加对应属性的变化的分支处理。localFieldId和fieldId是什么区别？？
                return onChangeViewModelTextStr((android.databinding.ObservableField<java.lang.String>) object, fieldId);
            case 1 :
                return onChangeViewModel((com.qdd.mvvmtest.MainViewModel) object, fieldId);
        }
        return false;
    }

    // 增加了对应属性的变化处理，fieldId判断为BR._all，标记dirtyFlags是什么作用？executeBindings时会根据dirtyFlags来更新脏掉的view。
    private boolean onChangeViewModelTextStr(android.databinding.ObservableField<java.lang.String> ViewModelTextStr, int fieldId) {
        if (fieldId == BR._all) {
            synchronized(this) {
                    mDirtyFlags |= 0x1L; // 表明textStr发生变化，需要更新。0x1L为textStr的标志位。
            }
            return true;
        }
        return false;
    }
    private boolean onChangeViewModel(com.qdd.mvvmtest.MainViewModel ViewModel, int fieldId) {
        if (fieldId == BR._all) {
            synchronized(this) {
                    mDirtyFlags |= 0x2L; // 从0x1L变为0x2L。
            }
            return true;
        }
        return false;
    }

    @Override
    protected void executeBindings() { // 该方法何时调用？？？？requestRebind的时候，以及LifecycleOwner（什么东西？？）调用onStart的时候
        long dirtyFlags = 0;
        synchronized(this) {
            dirtyFlags = mDirtyFlags;
            mDirtyFlags = 0;
        }
        // 执行绑定，增加了对应field的处理。
        java.lang.String viewModelTextStrGet = null;
        android.databinding.ObservableField<java.lang.String> viewModelTextStr = null;
        com.qdd.mvvmtest.MainViewModel viewModel = mViewModel;

        // 为什么判断0xbL????? 根据最下方的flag mapping,此范围出现脏块，都需要更新textStr。
        if ((dirtyFlags & 0xbL) != 0) {
                if (viewModel != null) {
                    // read viewModel.textStr
                    viewModelTextStr = viewModel.textStr;
                }
                updateRegistration(0, viewModelTextStr);


                if (viewModelTextStr != null) {
                    // read viewModel.textStr.get()
                    viewModelTextStrGet = viewModelTextStr.get();
                }
        }
        // batch finished
        if ((dirtyFlags & 0xbL) != 0) {
            // api target 1
            // TextViewBindingAdapter持有view对象，setText更新view的文字。
            android.databinding.adapters.TextViewBindingAdapter.setText(this.mboundView1, viewModelTextStrGet);
        }
    }
    // Listener Stub Implementations
    // callback impls
    // dirty flag
    private  long mDirtyFlags = 0xffffffffffffffffL;
    /* flag mapping
        flag 0 (0x1L): viewModel.textStr
        flag 1 (0x2L): viewModel
        flag 2 (0x3L): view
        flag 3 (0x4L): null
    flag mapping end*/
    //end
}
```
```java
// 看看这个TextViewBindingAdapter，就是设置view的属性，系统为我们提供了一些adapter，当我们需要给view自定义一些额外的属性时，可以自定义adapter，官方建议非迫不得已尽量不要自定义adapter。
@RestrictTo(RestrictTo.Scope.LIBRARY)
@SuppressWarnings({"WeakerAccess", "unused"})
@BindingMethods({
        @BindingMethod(type = TextView.class, attribute = "android:autoLink", method = "setAutoLinkMask"),
        @BindingMethod(type = TextView.class, attribute = "android:drawablePadding", method = "setCompoundDrawablePadding"),
        @BindingMethod(type = TextView.class, attribute = "android:editorExtras", method = "setInputExtras"),
        @BindingMethod(type = TextView.class, attribute = "android:inputType", method = "setRawInputType"),
        @BindingMethod(type = TextView.class, attribute = "android:scrollHorizontally", method = "setHorizontallyScrolling"),
        @BindingMethod(type = TextView.class, attribute = "android:textAllCaps", method = "setAllCaps"),
        @BindingMethod(type = TextView.class, attribute = "android:textColorHighlight", method = "setHighlightColor"),
        @BindingMethod(type = TextView.class, attribute = "android:textColorHint", method = "setHintTextColor"),
        @BindingMethod(type = TextView.class, attribute = "android:textColorLink", method = "setLinkTextColor"),
        @BindingMethod(type = TextView.class, attribute = "android:onEditorAction", method = "setOnEditorActionListener"),
})
public class TextViewBindingAdapter {

    private static final String TAG = "TextViewBindingAdapters";
    @SuppressWarnings("unused")
    public static final int INTEGER = 0x01;
    public static final int SIGNED = 0x03;
    public static final int DECIMAL = 0x05;

    @BindingAdapter("android:text")
    public static void setText(TextView view, CharSequence text) {
        final CharSequence oldText = view.getText();
        if (text == oldText || (text == null && oldText.length() == 0)) {
            return;
        }
        if (text instanceof Spanned) {
            if (text.equals(oldText)) {
                return; // No change in the spans, so don't set anything.
            }
        } else if (!haveContentsChanged(text, oldText)) {
            return; // No content changes, so don't set anything.
        }
        view.setText(text);
    }

    @InverseBindingAdapter(attribute = "android:text", event = "android:textAttrChanged")
    public static String getTextString(TextView view) {
        return view.getText().toString();
    }

    @BindingAdapter({"android:autoText"})
    public static void setAutoText(TextView view, boolean autoText) {
        KeyListener listener = view.getKeyListener();

        TextKeyListener.Capitalize capitalize = TextKeyListener.Capitalize.NONE;

        int inputType = listener != null ? listener.getInputType() : 0;
        if ((inputType & InputType.TYPE_TEXT_FLAG_CAP_CHARACTERS) != 0) {
            capitalize = TextKeyListener.Capitalize.CHARACTERS;
        } else if ((inputType & InputType.TYPE_TEXT_FLAG_CAP_WORDS) != 0) {
            capitalize = TextKeyListener.Capitalize.WORDS;
        } else if ((inputType & InputType.TYPE_TEXT_FLAG_CAP_SENTENCES) != 0) {
            capitalize = TextKeyListener.Capitalize.SENTENCES;
        }
        view.setKeyListener(TextKeyListener.getInstance(autoText, capitalize));
    }

    @BindingAdapter({"android:capitalize"})
    public static void setCapitalize(TextView view, TextKeyListener.Capitalize capitalize) {
        KeyListener listener = view.getKeyListener();

        int inputType = listener.getInputType();
        boolean autoText = (inputType & InputType.TYPE_TEXT_FLAG_AUTO_CORRECT) != 0;
        view.setKeyListener(TextKeyListener.getInstance(autoText, capitalize));
    }

    @BindingAdapter({"android:bufferType"})
    public static void setBufferType(TextView view, TextView.BufferType bufferType) {
        view.setText(view.getText(), bufferType);
    }

    @BindingAdapter({"android:digits"})
    public static void setDigits(TextView view, CharSequence digits) {
        if (digits != null) {
            view.setKeyListener(DigitsKeyListener.getInstance(digits.toString()));
        } else if (view.getKeyListener() instanceof DigitsKeyListener) {
            view.setKeyListener(null);
        }
    }

    @BindingAdapter({"android:numeric"})
    public static void setNumeric(TextView view, int numeric) {
        view.setKeyListener(DigitsKeyListener.getInstance((numeric & SIGNED) != 0,
                (numeric & DECIMAL) != 0));
    }

    @BindingAdapter({"android:phoneNumber"})
    public static void setPhoneNumber(TextView view, boolean phoneNumber) {
        if (phoneNumber) {
            view.setKeyListener(DialerKeyListener.getInstance());
        } else if (view.getKeyListener() instanceof DialerKeyListener) {
            view.setKeyListener(null);
        }
    }

    private static void setIntrinsicBounds(Drawable drawable) {
        if (drawable != null) {
            drawable.setBounds(0, 0, drawable.getIntrinsicWidth(), drawable.getIntrinsicHeight());
        }
    }

    @BindingAdapter({"android:drawableBottom"})
    public static void setDrawableBottom(TextView view, Drawable drawable) {
        setIntrinsicBounds(drawable);
        Drawable[] drawables = view.getCompoundDrawables();
        view.setCompoundDrawables(drawables[0], drawables[1], drawables[2], drawable);
    }

    @BindingAdapter({"android:drawableLeft"})
    public static void setDrawableLeft(TextView view, Drawable drawable) {
        setIntrinsicBounds(drawable);
        Drawable[] drawables = view.getCompoundDrawables();
        view.setCompoundDrawables(drawable, drawables[1], drawables[2], drawables[3]);
    }

    @BindingAdapter({"android:drawableRight"})
    public static void setDrawableRight(TextView view, Drawable drawable) {
        setIntrinsicBounds(drawable);
        Drawable[] drawables = view.getCompoundDrawables();
        view.setCompoundDrawables(drawables[0], drawables[1], drawable,
                drawables[3]);
    }

    @BindingAdapter({"android:drawableTop"})
    public static void setDrawableTop(TextView view, Drawable drawable) {
        setIntrinsicBounds(drawable);
        Drawable[] drawables = view.getCompoundDrawables();
        view.setCompoundDrawables(drawables[0], drawable, drawables[2],
                drawables[3]);
    }

    @BindingAdapter({"android:drawableStart"})
    public static void setDrawableStart(TextView view, Drawable drawable) {
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.JELLY_BEAN_MR1) {
            setDrawableLeft(view, drawable);
        } else {
            setIntrinsicBounds(drawable);
            Drawable[] drawables = view.getCompoundDrawablesRelative();
            view.setCompoundDrawablesRelative(drawable, drawables[1], drawables[2], drawables[3]);
        }
    }

    @BindingAdapter({"android:drawableEnd"})
    public static void setDrawableEnd(TextView view, Drawable drawable) {
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.JELLY_BEAN_MR1) {
            setDrawableRight(view, drawable);
        } else {
            setIntrinsicBounds(drawable);
            Drawable[] drawables = view.getCompoundDrawablesRelative();
            view.setCompoundDrawablesRelative(drawables[0], drawables[1], drawable, drawables[3]);
        }
    }

    @BindingAdapter({"android:imeActionLabel"})
    public static void setImeActionLabel(TextView view, CharSequence value) {
        view.setImeActionLabel(value, view.getImeActionId());
    }

    @BindingAdapter({"android:imeActionId"})
    public static void setImeActionLabel(TextView view, int value) {
        view.setImeActionLabel(view.getImeActionLabel(), value);
    }

    @BindingAdapter({"android:inputMethod"})
    @SuppressWarnings("ClassNewInstance")
    public static void setInputMethod(TextView view, CharSequence inputMethod) {
        try {
            Class<?> c = Class.forName(inputMethod.toString());
            view.setKeyListener((KeyListener) c.newInstance());
        } catch (ClassNotFoundException e) {
            Log.e(TAG, "Could not create input method: " + inputMethod, e);
        } catch (InstantiationException e) {
            Log.e(TAG, "Could not create input method: " + inputMethod, e);
        } catch (IllegalAccessException e) {
            Log.e(TAG, "Could not create input method: " + inputMethod, e);
        }
    }

    @BindingAdapter({"android:lineSpacingExtra"})
    public static void setLineSpacingExtra(TextView view, float value) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
            view.setLineSpacing(value, view.getLineSpacingMultiplier());
        } else {
            view.setLineSpacing(value, 1);
        }
    }

    @BindingAdapter({"android:lineSpacingMultiplier"})
    public static void setLineSpacingMultiplier(TextView view, float value) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
            view.setLineSpacing(view.getLineSpacingExtra(), value);
        } else {
            view.setLineSpacing(0, value);
        }
    }

    @BindingAdapter({"android:maxLength"})
    public static void setMaxLength(TextView view, int value) {
        InputFilter[] filters = view.getFilters();
        if (filters == null) {
            filters = new InputFilter[]{
                    new InputFilter.LengthFilter(value)
            };
        } else {
            boolean foundMaxLength = false;
            for (int i = 0; i < filters.length; i++) {
                InputFilter filter = filters[i];
                if (filter instanceof InputFilter.LengthFilter) {
                    foundMaxLength = true;
                    boolean replace = true;
                    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                        replace = ((InputFilter.LengthFilter) filter).getMax() != value;
                    }
                    if (replace) {
                        filters[i] = new InputFilter.LengthFilter(value);
                    }
                    break;
                }
            }
            if (!foundMaxLength) {
                // can't use Arrays.copyOf -- it shows up in API 9
                InputFilter[] oldFilters = filters;
                filters = new InputFilter[oldFilters.length + 1];
                System.arraycopy(oldFilters, 0, filters, 0, oldFilters.length);
                filters[filters.length - 1] = new InputFilter.LengthFilter(value);
            }
        }
        view.setFilters(filters);
    }

    @BindingAdapter({"android:password"})
    public static void setPassword(TextView view, boolean password) {
        if (password) {
            view.setTransformationMethod(PasswordTransformationMethod.getInstance());
        } else if (view.getTransformationMethod() instanceof PasswordTransformationMethod) {
            view.setTransformationMethod(null);
        }
    }

    @BindingAdapter({"android:shadowColor"})
    public static void setShadowColor(TextView view, int color) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
            float dx = view.getShadowDx();
            float dy = view.getShadowDy();
            float r = view.getShadowRadius();
            view.setShadowLayer(r, dx, dy, color);
        }
    }

    @BindingAdapter({"android:shadowDx"})
    public static void setShadowDx(TextView view, float dx) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
            int color = view.getShadowColor();
            float dy = view.getShadowDy();
            float r = view.getShadowRadius();
            view.setShadowLayer(r, dx, dy, color);
        }
    }

    @BindingAdapter({"android:shadowDy"})
    public static void setShadowDy(TextView view, float dy) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
            int color = view.getShadowColor();
            float dx = view.getShadowDx();
            float r = view.getShadowRadius();
            view.setShadowLayer(r, dx, dy, color);
        }
    }

    @BindingAdapter({"android:shadowRadius"})
    public static void setShadowRadius(TextView view, float r) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
            int color = view.getShadowColor();
            float dx = view.getShadowDx();
            float dy = view.getShadowDy();
            view.setShadowLayer(r, dx, dy, color);
        }
    }

    @BindingAdapter({"android:textSize"})
    public static void setTextSize(TextView view, float size) {
        view.setTextSize(TypedValue.COMPLEX_UNIT_PX, size);
    }

    private static boolean haveContentsChanged(CharSequence str1, CharSequence str2) {
        if ((str1 == null) != (str2 == null)) {
            return true;
        } else if (str1 == null) {
            return false;
        }
        final int length = str1.length();
        if (length != str2.length()) {
            return true;
        }
        for (int i = 0; i < length; i++) {
            if (str1.charAt(i) != str2.charAt(i)) {
                return true;
            }
        }
        return false;
    }

    @BindingAdapter(value = {"android:beforeTextChanged", "android:onTextChanged",
            "android:afterTextChanged", "android:textAttrChanged"}, requireAll = false)
    public static void setTextWatcher(TextView view, final BeforeTextChanged before,
            final OnTextChanged on, final AfterTextChanged after,
            final InverseBindingListener textAttrChanged) {
        final TextWatcher newValue;
        if (before == null && after == null && on == null && textAttrChanged == null) {
            newValue = null;
        } else {
            newValue = new TextWatcher() {
                @Override
                public void beforeTextChanged(CharSequence s, int start, int count, int after) {
                    if (before != null) {
                        before.beforeTextChanged(s, start, count, after);
                    }
                }

                @Override
                public void onTextChanged(CharSequence s, int start, int before, int count) {
                    if (on != null) {
                        on.onTextChanged(s, start, before, count);
                    }
                    if (textAttrChanged != null) {
                        textAttrChanged.onChange();
                    }
                }

                @Override
                public void afterTextChanged(Editable s) {
                    if (after != null) {
                        after.afterTextChanged(s);
                    }
                }
            };
        }
        final TextWatcher oldValue = ListenerUtil.trackListener(view, newValue, R.id.textWatcher);
        if (oldValue != null) {
            view.removeTextChangedListener(oldValue);
        }
        if (newValue != null) {
            view.addTextChangedListener(newValue);
        }
    }

    public interface AfterTextChanged {
        void afterTextChanged(Editable s);
    }

    public interface BeforeTextChanged {
        void beforeTextChanged(CharSequence s, int start, int count, int after);
    }

    public interface OnTextChanged {
        void onTextChanged(CharSequence s, int start, int before, int count);
    }
}
```