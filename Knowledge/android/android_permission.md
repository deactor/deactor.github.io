---
title:  "Android 权限"
---

# 简要记录修改恢复出厂设置删除SdCard(模拟的外部存储)数据过程
## 1. sdcard中的数据是哪块分区？
/sdcard是软链接，实际是/storage/emulated/0/，而/storage/emulated/0/和/data/media/0/数据是等同（实际测试是如此，但为什么？怎么做到的？）的。所以原生恢复
出厂格式化/data分区，会清除/storage数据。

## 2. 文件app是怎么删除sdcard中的数据的？
原生文件app,通过`DocumentsContract.deleteDocument()`方法删除文件，是跨进程的，根据实际log打印，最终FileSystemProvider运行deleteDocument方法是在com.android.externalstorage进程中。

```java
// DocumentsContract.java
public static boolean deleteDocument(@NonNull ContentResolver content, @NonNull Uri documentUri)
        throws FileNotFoundException {
    try {
        final Bundle in = new Bundle();
        in.putParcelable(DocumentsContract.EXTRA_URI, documentUri);

        content.call(documentUri.getAuthority(),
                METHOD_DELETE_DOCUMENT, null, in);
        return true;
    } catch (Exception e) {
        Log.w(TAG, "Failed to delete document", e);
        rethrowIfNecessary(e);
        return false;
    }
}

// FileSystemProvider.java
@Override
public void deleteDocument(String docId) throws FileNotFoundException {
    final File file = getFileForDocId(docId);
    final File visibleFile = getFileForDocId(docId, true);
    final boolean isDirectory = file.isDirectory();

    if (isDirectory) {
        FileUtils.deleteContents(file);
    }
    // We could be deleting pending media which doesn't have any content yet, so only throw
    // if the file exists and we fail to delete it.
    if (file.exists() && !file.delete()) {
        throw new IllegalStateException("Failed to delete " + file);
    }

    onDocIdChanged(docId);
    onDocIdDeleted(docId);
    removeFromMediaStore(visibleFile);
}
```
## 3. 删除sdcard中的数据所遇到的权限问题
因为不走原生恢复出厂的格式化逻辑，需要在System_server进程中删除sdcard中的数据，结果没有权限，通过`File.listFiles()`方法获取不到文件夹下的文件。考虑两种方案：
+ 方案一： 找一个有权限的进程来删
+ 方案二： 给system_server添加对应权限

由于恢复出厂设置运行后要进行重启，如果采用方案一，则需要进行进程间交互，确保拿到删除结果后再执行重启，故考虑优先使用方案二。 
根据`ls -l /storage/emulated/0`下的文件，存在几种不同的user和group。（**此处应该有图补充说明**）我们只要给system_server添加到对应的用户组上就可以了。

+ 首先，查看groupid的String对应的int.其定义在system/core/include/private/android_filesystem_config.h文件中（**此处应该有图补充说明**）。  
+ 其次，查看system_server进程当前是在哪些组中，ps获取到system_server的pid,执行`cat /proc/pid/status`,输出的信息中groups信息即system_server所在的组（**此处应该有图补充说明**）。  
+ 最后，根据查看到externalstorage进程属于哪些组，同样也将system_server加到这些组中。如何加，看下面


>注意：/storage/emulated/0/Music这种文件夹，其user和group都是com.android.providers.media.module进程的用户id, 并不需要将system_server加到providers用户组中，因为发现externalstorage进程可以删除此处文件，那么必定不是要providers的用户id。最终发现给进程加上AID_EVERYBODY用户组则可以访问这些文件夹（但是为什么？详细原理待总结）。    

>注意：加上AID_EVERYBODY可以操作/storage/emulated/0/Music,但是不能操作/data/media/0/Music，虽然两处的数据是同一块，但是访问权限却不一样。（具体原因待总结）

## 4. 进程添加权限组的几种类型
### 1）应用进程
应用进程的启动，其groups是在ProcessList.java中startProcessLocked时会判断需要给对应进程什么样的groups.之后在zygote进行进程fork时作为`--setgroups`的参数。
```java
// ProcessList.java
private int[] computeGidsForProcess(int mountExternal, int uid, int[] permGids) {
    ...

    if (mountExternal == Zygote.MOUNT_EXTERNAL_ANDROID_WRITABLE
            || mountExternal == Zygote.MOUNT_EXTERNAL_PASS_THROUGH) {
            
        gidList.add(UserHandle.getUid(UserHandle.getUserId(uid), Process.SDCARD_RW_GID));

        gidList.add(Process.EXT_DATA_RW_GID);
        gidList.add(Process.EXT_OBB_RW_GID);
    }
    if (mountExternal == Zygote.MOUNT_EXTERNAL_INSTALLER) {
        gidList.add(Process.EXT_OBB_RW_GID);
    }
    if (mountExternal == Zygote.MOUNT_EXTERNAL_PASS_THROUGH) {
        gidList.add(Process.MEDIA_RW_GID);
    }

    ...
}
```
### 2）System_server
ZygoteInit时fork进程，添加的固定参数。
```java
// ZygoteInit.java
private static Runnable forkSystemServer(String abiList, String socketName,
        ZygoteServer zygoteServer) {
    ...

    String args[] = {
            "--setuid=1000",
            "--setgid=1000",
            "--setgroups=1001,1002,1003,1004,1005,1006,1007,1008,1009,1010,1018,1021,1023,"
                    + "1024,1032,1065,3001,3002,3003,3006,3007,3009,3010,3011,"
                    + "1015,1077,1078,1079,9997",
            "--capabilities=" + capabilities + "," + capabilities,
            "--nice-name=system_server",
            "--runtime-args",
            "--target-sdk-version=" + VMRuntime.SDK_VERSION_CUR_DEVELOPMENT,
            "com.android.server.SystemServer",
    };
    ZygoteArguments parsedArgs = null;

    int pid;

    try {
        parsedArgs = new ZygoteArguments(args);
        ...
        pid = Zygote.forkSystemServer(
                parsedArgs.mUid, parsedArgs.mGid,
                parsedArgs.mGids,
                parsedArgs.mRuntimeFlags,
                null,
                parsedArgs.mPermittedCapabilities,
                parsedArgs.mEffectiveCapabilities);
    } catch (IllegalArgumentException ex) {
        throw new RuntimeException(ex);
    }
    ...
}
```
### 3）Linux进程
Linux由init进程解析rc文件进行启动，rc文件中可以配置group参数，比如vold.rc中进程的启动配置
```ini
service vold /system/bin/vold \                                                                                                            
        --blkid_context=u:r:blkid:s0 --blkid_untrusted_context=u:r:blkid_untrusted:s0 \
        --fsck_context=u:r:fsck:s0 --fsck_untrusted_context=u:r:fsck_untrusted:s0
    class core
    ioprio be 2
    writepid /dev/cpuset/foreground/tasks
    shutdown critical
    group root reserved_disk
```