---
title:  "分析AudioTrack的usage和streamtype参数"
---
#### 抓取方式
在对应媒体播放时执行：
```
dumpsys media.audio_flinger
```
#### 日志信息
根据应用进程号找到session id，如这里蓝牙的session id是25
```
Clients:
  pid: 5890
Notification Clients:
   pid    uid  name
   245   1041  audioserver
   867   1000  android.uid.system
  5890 1001002  android.uid.bluetooth
Global session refs:
  session  cnt     pid    uid  name
       25    1    5890 1001002  android.uid.bluetooth
```
再根据session id找到如下信息：
```
  Stream volumes in dB: 0:0, 1:-6, 2:0, 3:0, 4:0, 5:0, 6:-inf, 7:-6, 8:-6, 9:0, 10:0, 11:0, 12:0, 13:0, 14:0
  Normal mixer raw underrun counters: partial=0 empty=0
  1 Tracks of which 1 are active
    Type     Id Active Client Session Port Id S  Flags   Format Chn mask  SRate ST Usg CT  G db  L dB  R dB  VS dB   Server FrmCnt  FrmRdy F Underruns  Flushed   Latency
             57    yes   5890      25      38 A  0x001 00000003 00000003  96000  4   4  2     0     0     0     0  00547082   3848    1928 A    192844        0   95.39 t
  0 Effect Chains
```
其中Usg就是Usage，ST为StreamType。该dump信息是在`frameworks/av/services/audioflinger/Tracks.cpp`中创建，`... ST Usg CT ...`是在`appendDumpHeader方法`中，下方具体的值是在`appendDump`方法中