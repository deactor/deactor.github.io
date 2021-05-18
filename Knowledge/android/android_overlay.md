---
title:  "Android Overlay换肤"
---
### Overlay配置要点：
参考[官网](https://source.android.google.cn/devices/architecture/rros)进行配置，Android 11之后会有一些新的配置方式，此文设置是在Android 9上的经验。

1. 新建Module，将代码全部删除，只保留资源文件，资源的name要和目标app的资源名一致。

2. 使用和目标app相同的签名，AndroidMainfest中system uid并不需要一致。

3. AndroidMainfest.xml配置  
    ```
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.example.overlay">
        <application android:hasCode="false" />
        <overlay android:targetPackage="com.example.target"
                 android:isStatic="false"
                 android:priority="5"/>
    </manifest>
    ```   
   + 动态换肤要点：android:isStatic="false"  
   + 要覆盖的目标app包名：android:targetPackage="com.example.target"

4. 编译生成的apk就是皮肤包。安装后，设置该皮肤包enable即可实现换肤。（安装就行，并不需要像其他blog中写的要放到指定目录下，放指定目录下一般是系统Rom集成时放皮肤包用的，用于开机时加载安装皮肤包。）

### 主要原理：
1. AndroidMainfest.xml中携带<overlay>标签的apk在安装时，会被识别为皮肤包，包解析后会存储相应信息到OverlaySetting中，并由OverlayManagerService处理。Overlay的信息会被存储在/data/system/overlays.xml中。
2. 在安装时，OverlayManagerService还会通过idmap建立皮肤中资源id到目标包资源id的映射，会在**/data/resource-cache**目录下生成一个idmap文件，所以两个apk中Resource.src中的资源id不一样没关系，因为是有映射的，但是资源名一定要一样，不然没法映射。
3. overlays.xml中存储了各皮肤包的状态（enable、disable），代码位置等信息。
4. 通过OverlayManagerService进行setEnable对应皮肤包包名来使对应皮肤包生效。之后查找资源就先从enable的皮肤包中查找资源。
   

>setEnable  
>这个操作并不一定要在目标apk中进行，在任何apk中都可以，比如在Settings中切换主题，获取此处的皮肤包包名（OverlayManagerService可以获取到所有已安装的皮肤包信息），将其设置为enable。  
>OverlayManagerService并不是开放的，所以非源码环境并不能获取到该Service。第三方只能通过shell命令进行enable，或Rom进行了客制化，有相应的接口，比如Settings中规定皮肤包名尾部为overlay1，overlay2等，当在settings中切换主题时，就将所有包名尾部是overlay1的包进行enable。而且请求换主题的app需要拥有android.permission.CHANGE_OVERLAY_PACKAGES权限，而且还要是System或Root的uid，比如这里请求换主题的app就是Settings。  
>setEnable在OverlayManagerService中只会enable目标app的一个皮肤包，比如目标app有overlay1，overlay2，overlay3这三个皮肤包，当enable overlay1时，自动把overlay2，overlay3都设置为disable了。

Overlays.xml中的信息如下：
```
<overlays version="4">
    <item packageName="com.android.theme.icon_pack.kai.settings" userId="0" targetPackageName="com.android.settings" baseCodePath="/product/overlay/IconPackKaiSettings/IconPackKaiSettingsOverlay.apk" state="2" isEnabled="false" isStatic="false" priority="2147483647" category="android.theme.customization.icon_pack.settings" />
    <item packageName="com.android.theme.icon_pack.sam.settings" userId="0" targetPackageName="com.android.settings" baseCodePath="/product/overlay/IconPackSamSettings/IconPackSamSettingsOverlay.apk" state="2" isEnabled="false" isStatic="false" priority="2147483647" category="android.theme.customization.icon_pack.settings" />
</overlays>
```

### overlay常用的指令：
#### 查看当前overlay包状态
```
   adb shell cmd overlay list
```
该指令会列出目标包及其拥有的皮肤包，信息如下：
```
com.android.systemui
[x] com.android.systemui.auto_generated_rro_vendor__
[ ] com.android.theme.icon_pack.rounded.systemui
[ ] com.android.theme.icon_pack.victor.systemui
[ ] com.android.theme.icon_pack.kai.systemui
[ ] com.android.theme.icon_pack.sam.systemui
[ ] com.android.theme.icon_pack.filled.systemui
[ ] com.android.theme.icon_pack.circular.systemui
```
前面有x的代表时当前enable的皮肤包。

#### enable指定皮肤包
```
    adb shell cmd overlay enable --user 0 com.example.overlay
```
该指令就是enable开头例子中的com.example.overlay的这个皮肤包。之后其目标包com.example.target这个apk的皮肤就被替换了。


#### 查看idmap文件信息
下面这个指令是Android 9上用的，信息并不能直观看出映射关系。
```
    adb shell idmap --inspect /data/resource-cache/vendor@overlay@SystemUI__auto_generated_rro_vendor.apk@idmap
```
Android 11上使用idmap2，可以直接看到映射关系。
```
    adb shell idmap2 dump --idmap-path /data/resource-cache/vendor@overlay@SystemUI__auto_generated_rro_vendor.apk@idmap
```
输出的信息如下，就很直观：
```
Paths:
    target apk path  : /system_ext/priv-app/SystemUI/SystemUI.apk
    overlay apk path : /vendor/overlay/SystemUI__auto_generated_rro_vendor.apk
Mapping:
    0x7f03002b -> 0x7f010000 array/config_doze_brightness_sensor_to_brightness
    0x7f03002c -> 0x7f010001 array/config_doze_brightness_sensor_to_scrim_opacity
    0x7f050014 -> 0x7f020000 bool/config_hspa_data_distinguishable
    0x7f050033 -> 0x7f020001 bool/doze_display_state_supported
    0x7f050034 -> 0x7f020002 bool/doze_double_tap_reports_touch_coordinates
    0x7f050038 -> 0x7f020004 bool/doze_suspend_display_state_supported
    0x7f0703d9 -> 0x7f030000 dimen/rounded_corner_content_padding
    0x7f07043f -> 0x7f030001 dimen/status_bar_padding_top
    0x7f0805a4 -> 0x7f040000 drawable/rounded
    0x7f0b000d -> 0x7f050000 integer/config_keyguardRefreshRate
    0x7f0b000e -> 0x7f050001 integer/config_lockScreenDisplayTimeout
    0x7f110222 -> 0x7f060000 string/config_rounded_mask
    0x7f1102b2 -> 0x7f060001 string/doze_brightness_sensor_type
```

### 碰到的坑
1. 目标包不能使用AppComptActivity，只能用Activity，不然换肤后会直接奔溃，报这个Activity需要设置AppCompt主题，但是在AndroidManifest中给这个activity配置了对应的主题也没用，就是奔溃。原因不明。
2. 不能覆盖drawable下的xml文件，报错的那种，比如drawable下放一个帧动画的xml，那么设置皮肤包后，目标app在用到这个xml的时候就会报找不到资源，所以要在皮肤包中将这个xml删掉，只保留对应的图片文件就行。