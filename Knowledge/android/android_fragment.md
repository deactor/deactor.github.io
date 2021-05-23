---
title:  "Android Fragment相关"
---
### 参考
[官网](https://developer.android.com/guide/fragments/fragmentmanager)

### 常用摘要
#### 添加到Activity
FragmentContainerView用来承载Fragment。
```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.fragment.app.FragmentContainerView
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/fragment_container_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:name="com.qdd.test.testfragment.ExampleFragment" />
        <!-- name属性指明要实例化的fragment. 在activity布局inflated时，该fragment被实例化，fragment的onInflate()被回调，
        FragmentTransaction被创建并添加该fragment到FragmentManager中-->
        <!-- 如果不知指明，可以在activity通过FragmentManager和FragmentTransaction来添加，替换，移除fragment -->

</androidx.constraintlayout.widget.ConstraintLayout>
```

#### Fragment管理及通信
1. 通过FragmentManager管理，添加，移除，替换都通过FragmentTransaction事务进行处理。  
2. Activity中可以通过getSupportFragmentManager()获取到FragmentManager。  
3. Fragment可以嵌套子Fragment，父Fragment可通过getChildFragmentManager()获取管理子Fragment的FragmentManager，子Fragment也可以通过getParentFragmentManager()，获取其宿主FragmentManager.  
   
```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    if (savedInstanceState == null) {
        // bundle给Fragment传递数据
        Bundle bundle = new Bundle();
        bundle.putInt("some_int", 10);

        // 获取FragmentManager
        FragmentManager fragmentManager = getSupportFragmentManager();

         // FragmentTransaction addToBackStack() popBackStack()
         // 不addBackStack，FragmentTransaction的remove操作会销毁Fragment实例，如果addBackStack则只是标记STOP状态，pop仍能回到该Fragment。
         // setReorderingAllowed(true) 可优化事务中涉及的 Fragment 的状态变化，以使动画和过渡正常运行。
         getSupportFragmentManager().beginTransaction()
                 .setReorderingAllowed(true)
                 .add(R.id.fragment_container_view, ExampleFragment.class, bundle)
                 .commit();

        // 上面的方式是通过FragmentManager内部的FragmentFactory来实例化ExampleFragment，默认通过反射调用无参构造方法进行实例化，也可以自定义FragmentFactory。
        // 也可以直接传递ExampleFragment对象。
        ExampleFragment exampleFragment = new ExampleFragment();
        getSupportFragmentManager().beginTransaction()
                .setReorderingAllowed(true)
                .add(R.id.fragment_container_view, exampleFragment)
                .commit();

        // 设置结果监听器，可以接收Fragment setFragmentResult传递的结果。
        fragmentManager.setFragmentResultListener("abc", this, new FragmentResultListener() {
            @Override
            public void onFragmentResult(@NonNull String requestKey, @NonNull Bundle result) {
                Log.i(TAG, "onFragmentResult: requestKey -> requestKey : " + requestKey);
            }
        });

        // 设置结果监听器，可以接收Fragment setFragmentResult传递的结果。
        fragmentManager.setFragmentResultListener("abc", this, new FragmentResultListener() {
            @Override
            public void onFragmentResult(@NonNull String requestKey, @NonNull Bundle result) {
                Log.i(TAG, "onFragmentResult: requestKey -> requestKey : " + requestKey);
            }
        });
    }
}
```

```
public class ExampleFragment extends Fragment {
    public ExampleFragment() {
        super(R.layout.example_fragment);
    }

    // Fragment中可以嵌套Fragment，父Fragment可通过getChildFragmentManager()获取管理子Fragment的FragmentManager
    // 子Fragment也可以通过getParentFragmentManager()，获取其宿主FragmentManager.

    @Override
    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        // 接收activity传过来的数据
        int someInt = requireArguments().getInt("some_int");
        getParentFragmentManager().setFragmentResult("abc",new Bundle());
    }
}
```