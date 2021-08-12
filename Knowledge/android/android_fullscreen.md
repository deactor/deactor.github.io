## 全屏的三种模式
模式：
+ 向后倾斜模式
+ 沉浸模式
+ 粘性沉浸模式

三种模式下，系统栏都是隐藏的，Activity会接到触摸事件，区别在于系统栏重新显示出来的方式。

### 向后倾斜模式
应用场景：适用于用户不会与屏幕进行大量互动的全屏体验，例如在观看视频时。

退出方式：点按屏幕上的任意位置。

实现方式：调用 setSystemUiVisibility() 并传递 SYSTEM_UI_FLAG_FULLSCREEN
和 SYSTEM_UI_FLAG_HIDE_NAVIGATION
```java
setSystemUiVisibility(View.SYSTEM_UI_FLAG_FULLSCREEN
                | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION);
```

### 沉浸模式
应用场景：适用于用户将与屏幕进行大量互动的应用。示例包括游戏、查看图库中的图片或者阅读分页内容，如图书或演示文稿中的幻灯片。

退出方式：从隐藏系统栏的任一边滑动。

实现方式：调用 setSystemUiVisibility() 并将 SYSTEM_UI_FLAG_IMMERSIVE 标志与 SYSTEM_UI_FLAG_FULLSCREEN 和 SYSTEM_UI_FLAG_HIDE_NAVIGATION 一起传递。
```java
setSystemUiVisibility(SYSTEM_UI_FLAG_IMMERSIVE
                | View.SYSTEM_UI_FLAG_FULLSCREEN
                | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION);
```

> 注意：触发退出的这个手势，app是接不到的。app可以接收到隐藏和显示的回调。

### 粘性沉浸模式
应用场景：需要沉浸模式且需要接收用户从屏幕边缘滑动的手势时。
>例如，在使用这种方法的绘图应用中，如果用户想绘制从屏幕最边缘开始的线条，则从这个边缘滑动会显示系统栏，同时还会开始绘制从最边缘开始的线条。

退出方式：从隐藏系统栏的任一边滑动。
> 注意：无互动几秒钟后，或者用户在系统栏之外的任何位置轻触或做手势时，系统栏会自动消失。

实现方式：调用 setSystemUiVisibility() 并将
SYSTEM_UI_FLAG_IMMERSIVE_STICKY 标志与 SYSTEM_UI_FLAG_FULLSCREEN 和
SYSTEM_UI_FLAG_HIDE_NAVIGATION 一起传递。
```java
setSystemUiVisibility(SYSTEM_UI_FLAG_IMMERSIVE_STICKY 
                | View.SYSTEM_UI_FLAG_FULLSCREEN
                | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION);
```
> 注意：触发退出的手势，app可以接到，但是app接不到隐藏和显示的回调。

## SYSTEM UI FLAG
### SYSTEM_UI_FLAG_LOW_PROFILE
作用：调暗状态栏和导航栏，状态栏和导航栏上的图标会消失。  
退出方式：用户轻触屏幕的状态栏或导航栏区域

### SYSTEM_UI_FLAG_FULLSCREEN
作用：隐藏状态栏，同时app自身的ActionBar也会隐藏。  
退出方式：见上面的三种模式。

### SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
作用：允许app内容显示在状态栏下方，并不会隐藏状态栏。通常与SYSTEM_UI_FLAG_LAYOUT_STABLE配合使用以保证app内容大小不会随着状态栏的隐藏和显示发生调整。
> 注意：使用此方法时，需要确保应用界面中的关键部分（例如，地图应用中的内置控件）不会被系统栏覆盖，否则会导致应用无法使用。在大多数情况下，要处理这个问题，可以通过向
> XML 布局文件添加 android:fitsSystemWindows 属性并设置为
> true。这会调整父级 ViewGroup
> 的内边距，为系统窗口留出空间。这对于大多数应用来说已经足够。

### SYSTEM_UI_FLAG_HIDE_NAVIGATION
作用：隐藏导航栏
退出方式：见上面的三种模式。

### SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
作用：让内容显示在导航栏后面。注意事项等同SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN。

### SYSTEM_UI_FLAG_LAYOUT_STABLE
作用：帮助app保持稳定布局。

## 响应界面可见度变更
```java
View decorView = getWindow().getDecorView();
// 注册监听
decorView.setOnSystemUiVisibilityChangeListener
        (new View.OnSystemUiVisibilityChangeListener() {
    @Override
    public void onSystemUiVisibilityChange(int visibility) {
        // Note that system bars will only be "visible" if none of the
        // LOW_PROFILE, HIDE_NAVIGATION, or FULLSCREEN flags are set.
        if ((visibility & View.SYSTEM_UI_FLAG_FULLSCREEN) == 0) {
            // TODO: The system bars are visible. Make any desired
            // adjustments to your UI, such as showing the action bar or
            // other navigational controls.
        } else {
            // TODO: The system bars are NOT visible. Make any desired
            // adjustments to your UI, such as hiding the action bar or
            // other navigational controls.
        }
    }
});
```
