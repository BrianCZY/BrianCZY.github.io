---
layout: post
title: Android 6.0 开放root权限，app获取root权限
categories: Android
description: 使用Logger时，不在本地生成日志文件的问题梳理
keywords: Android， 6.0， 源码，root
---

<!--注：本文更新于2019-11-5-->

本次修改的是MTK 6.0版本系统源码，已经测试可以实现app 获得root权限

具体修改思路参照 ：https://blog.csdn.net/alianqiugui/article/details/79392101

1、修改app_main.cpp 文件

```shell
vim frameworks/base/cmds/app_process/app_main.cpp  +185
```

注释掉prctl(PR_SET_NO_NEW_PRIVS,1,0,0,0) 这部分代码，主要是用来防止动态的改变权限，跟SELinux有关，具体修改如下：

```java
186
187 int main(int argc, char* const argv[])
188 {
189     /*if (prctl(PR_SET_NO_NEW_PRIVS, 1, 0, 0, 0) < 0) {
190         // Older kernels don't understand PR_SET_NO_NEW_PRIVS and return
191         // EINVAL. Don't die on such kernels.
192         if (errno != EINVAL) {
193             LOG_ALWAYS_FATAL("PR_SET_NO_NEW_PRIVS failed: %s", strerror(errno));
194             return 12;
195         }
196     }*/
197
198     AppRuntime runtime(argv[0], computeArgBlockSize(argc, argv));



```



2、修改com_android_internal_os_Zygote.cpp文件

```she
vim frameworks/base/core/jni/com_android_internal_os_Zygote.cpp  +219
```

注释掉DropCapabilitiesBoundingSet 方法里面的代码，修改如下：

```java
219 static void DropCapabilitiesBoundingSet(JNIEnv* env) {
220   /*for (int i = 0; prctl(PR_CAPBSET_READ, i, 0, 0, 0) >= 0; i++) {
221     int rc = prctl(PR_CAPBSET_DROP, i, 0, 0, 0);
222     if (rc == -1) {
223       if (errno == EINVAL) {
224         ALOGE("prctl(PR_CAPBSET_DROP) failed with EINVAL. Please verify "
225               "your kernel is compiled with file capabilities support");
226       } else {
227         ALOGE("prctl(PR_CAPBSET_DROP) failed");
228         RuntimeAbort(env);
229       }
230     }
231   }*/
232 }


```



3、修改su.c

```shell
vim system/extras/su/su.c  +111
```



注释main函数的这部分代码，修改为：

```java
111    /* if (myuid != AID_ROOT && myuid != AID_SHELL) {
112         fprintf(stderr,"su: uid %d not allowed to su\n", myuid);
113         return 1;
114     }*/
115

```



4、修改 /system/xbin/su 文件的权限&分组

查找资料得知6.0修改的文件为fs_config.c , 但由于我这个版本并没有找到fs_config.c文件

文件位置为：/**system**/core/libcutils/fs_config.c ，所以需要搜索一下具体位置，通常配置

文件的权限信息的关键是 android_files ，所有搜索android_files查找配置的具体文件。

（1）、查找配置权限的具体文件 

命令：grep -nir AID_SHELL  * 
system/core/include/private/android_filesystem_config.h
命令解析：在当前目录及子目录搜索所有文件，并列出包含AID_SHELL内容 的文件及行号 

```shell
ubuntu@ubuntu-Pc:~/phone/mtk/kt109/alps/system/core$ grep -nir android_files *
adb/adb.c:43:#include <private/android_filesystem_config.h>
adb/file_sync_service.c:33:#include <private/android_filesystem_config.h>
cpio/mkbootfs.c:15:#include <private/android_filesystem_config.h>
debuggerd/tombstone.cpp:39:#include <private/android_filesystem_config.h>
debuggerd/debuggerd.cpp:41:#include <private/android_filesystem_config.h>
fs_mgr/fs_mgr_verity.c:32:#include <private/android_filesystem_config.h>
fs_mgr/fs_mgr.c:38:#include <private/android_filesystem_config.h>
healthd/BatteryPropertiesRegistrar.cpp:24:#include <private/android_filesystem_config.h>
include/private/android_filesystem_capability.h:21:#ifndef _SYSTEM_CORE_INCLUDE_PRIVATE_ANDROID_FILESYSTEM_CAPABILITY_H
include/private/android_filesystem_capability.h:22:#define _SYSTEM_CORE_INCLUDE_PRIVATE_ANDROID_FILESYSTEM_CAPABILITY_H
include/private/android_filesystem_config.h:27:#ifndef _ANDROID_FILESYSTEM_CONFIG_H_
include/private/android_filesystem_config.h:28:#define _ANDROID_FILESYSTEM_CONFIG_H_
include/private/android_filesystem_config.h:38:#include "android_filesystem_capability.h"
include/private/android_filesystem_config.h:230:static const struct fs_path_config android_files[] = {
include/private/android_filesystem_config.h:290:    pc = dir ? android_dirs : android_files;
init/init.c:51:#include <private/android_filesystem_config.h>
init/ueventd.c:26:#include <private/android_filesystem_config.h>
init/util.c:43:#include <private/android_filesystem_config.h>
init/builtins.c:54:#include <private/android_filesystem_config.h>
init/devices.c:39:#include <private/android_filesystem_config.h>
init/property_service.c:47:#include <private/android_filesystem_config.h>
libcutils/sockets.c:22:#include <private/android_filesystem_config.h>
liblog/logd_write.c:46:#include <private/android_filesystem_config.h>
libprocessgroup/processgroup.cpp:31:#include <private/android_filesystem_config.h>
logd/LogCommand.cpp:21:#include <private/android_filesystem_config.h>
logd/LogStatistics.cpp:22:#include <private/android_filesystem_config.h>
logd/main.cpp:32:#include "private/android_filesystem_config.h"
logd/LogBuffer.h:26:#include <private/android_filesystem_config.h>
logd/CommandListener.cpp:29:#include <private/android_filesystem_config.h>
logwrapper/logwrap.c:32:#include "private/android_filesystem_config.h"
run-as/package.c:22:#include <private/android_filesystem_config.h>
run-as/run-as.c:33:#include <private/android_filesystem_config.h>
sdcard/sdcard.c:49:#include <private/android_filesystem_config.h>


```

（2）修改system/xbin/su 权限及分组

```shell
vim system/core/include/private/android_filesystem_config.h   +230
```

具体修改为：

```java
 static const struct fs_path_config android_files[] = {
232 	{ 00750, AID_ROOT,      AID_ROOT,     0, "system/etc/partition_permission.sh" },
233     { 00440, AID_ROOT,      AID_SHELL,     0, "system/etc/init.goldfish.rc" },
234     { 00550, AID_ROOT,      AID_SHELL,     0, "system/etc/init.goldfish.sh" },
235     { 00440, AID_ROOT,      AID_SHELL,     0, "system/etc/init.trout.rc" },
236     { 00550, AID_ROOT,      AID_SHELL,     0, "system/etc/init.ril" },
237     { 00550, AID_ROOT,      AID_SHELL,     0, "system/etc/init.testmenu" },
238     { 00550, AID_DHCP,      AID_SHELL,     0, "system/etc/dhcpcd/dhcpcd-run-hooks" },
239     { 00550, AID_DHCP,      AID_SHELL,     0, "system/etc/wide-dhcpv6/dhcp6c.script" },
240     { 00444, AID_RADIO,     AID_AUDIO,     0, "system/etc/AudioPara4.csv" },
241     { 00555, AID_ROOT,      AID_ROOT,      0, "system/etc/ppp/*" },
242     { 00555, AID_ROOT,      AID_ROOT,      0, "system/etc/rc.*" },
243     { 00644, AID_SYSTEM,    AID_SYSTEM,    0, "data/app/*" },
244     { 00644, AID_MEDIA_RW,  AID_MEDIA_RW,  0, "data/media/*" },
245     { 00644, AID_SYSTEM,    AID_SYSTEM,    0, "data/app-private/*" },
246     { 00644, AID_APP,       AID_APP,       0, "data/data/*" },
247     { 00755, AID_ROOT,      AID_ROOT,      0, "system/bin/ping" },
248
249     /* the following file is INTENTIONALLY set-gid and not set-uid.
250      * Do not change. */
251     { 02750, AID_ROOT,      AID_INET,      0, "system/bin/netcfg" },
252
253     /* the following five files are INTENTIONALLY set-uid, but they
254      * are NOT included on user builds. */
255     { 06755, AID_ROOT,      AID_ROOT,      0, "system/xbin/su" },  //修改本行
256     { 06755, AID_ROOT,      AID_ROOT,      0, "system/xbin/librank" },
257     { 06755, AID_ROOT,      AID_ROOT,      0, "system/xbin/procrank" },
258     { 06755, AID_ROOT,      AID_ROOT,      0, "system/xbin/procmem" },
259     { 00755, AID_ROOT,      AID_ROOT,      0, "system/xbin/tcpdump" },
260     { 04770, AID_ROOT,      AID_RADIO,     0, "system/bin/pppd-ril" },
261

```



5.禁止selinux

禁止selinux有多种方式，这里是在selinux_is_disabled函数中修改，具体文件是system/core/init.cpp ，

但是本版本源码也找不到 system/core/init.cpp 文件，全局搜索一下 grep -nir selinux_is_disabled *  ，

得知在 system/core/init/init.c 文件的949行



```shell
ubuntu@ubuntu-Pc:~/phone/mtk/kt109/alps/system/core$ grep -nir selinux_is_disabled *
init/init.c:949:static bool selinux_is_disabled(void)
init/init.c:995:    if (selinux_is_disabled()) {
init/init.c:1074:    if (selinux_is_disabled()) {
ubuntu@ubuntu-Pc:~/phone/mtk/kt109/alps/system/core$ cd init/
```



修改system/core/init/init.c  文件为：

```java
 static bool selinux_is_disabled(void)
  {
  #ifdef ALLOW_DISABLE_SELINUX

 if (access("/sys/fs/selinux", F_OK) != 0) {
      /* SELinux is not compiled into the kernel, or has been disabled
       * via the kernel command line "selinux=0".
       */
      return true;
  }
 
 if ((property_get("ro.boot.selinux", tmp) != 0) && (strcmp(tmp, "disabled") == 0)) {
      /* SELinux is compiled into the kernel, but we've been told to disable it. */
     return true;
 }
  #endif

 // return false;
	return true;    //修改这里 将return false 改为 return true
  }
```





6、User版本的修改需要修改下面两个文件，如果编译debug版本，那么可以跳过本步骤

（1）修改mk，使得所有版本编译时也编译su

```shell
 vim system/extras/su/Android.mk
```

将 LOCAL_MODULE_TAGS := debug  修改为：

LOCAL_MODULE_TAGS := optional

```shell
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)

LOCAL_SRC_FILES:= su.c

LOCAL_MODULE:= su

LOCAL_FORCE_STATIC_EXECUTABLE := true

LOCAL_STATIC_LIBRARIES := libc

LOCAL_MODULE_PATH := $(TARGET_OUT_OPTIONAL_EXECUTABLES)
//LOCAL_MODULE_TAGS := debug
LOCAL_MODULE_TAGS := optional

include $(BUILD_EXECUTABLE)


```



（2）添加su的编译目录

```shell
 vim build/target/product/core.mk
```

添加 ：su \

修改后的文件如下:

```shell
PRODUCT_PACKAGES += \
    su \
    BasicDreams \
    Browser \
    Calculator \
    Calendar \
    CalendarProvider \
    CaptivePortalLogin \
    CertInstaller \
    DeskClock \
    DocumentsUI \
    DownloadProviderUi \
    ExternalStorageProvider \
    FusedLocation \
    InputDevices \
    KeyChain \
    Keyguard \
    Launcher2 \
    ManagedProvisioning \
    PicoTts \
    PacProcessor \
    libpac \
    PrintSpooler \
    ProxyHandler \
    Settings \
    SharedStorageBackup \
    Telecom \
    TeleService \
    VpnDialogs \
    MmsService

```

7、编译源码

​		 source build/envsetup.sh

​		 lunch   //并选择编译的版本，我选择的是user 版本

​		make -j8 

8、烧录到开发板上

9、查看及测试权限



（1）查看 /system/xbin/su 的权限

进入adb，执行 ls -l /system/xbin/su   ，从结果来看，su 属于root分组并且对于普通用户具有执行权限

```shell
C:\Users\brian>adb shell
shell@JTY:/ $ ls -l /system/xbin/su
-rwsr-sr-x root     root       276504 2019-11-05 14:07 su
shell@JTY:/ $
```



（2）测试root权限

运行一个任意app, 进入adb ，并根据该包名搜索具体的进程信息。

（下面以系统内置的浏览器为例 ，其包名为： com.android.browser）

```shell
C:\Users\brian>adb shell
shell@JTY:/ $ ps | grep com.android.browser      //查找进程
u0_a19    2540  310   1326428 48224 ffffffff 00000000 S com.android.browser
shell@JTY:/ $ su u0_a19   //切换到app进程
shell@JTY:/ $ id
uid=10019(u0_a19) gid=10019(u0_a19) groups=1003(graphics),1004(input),1007(log),1011(adb),1015(sdcard_rw),1028(sdcard_r),3001(net_bt_admin),3002(net_bt),3003(inet),3006(net_bw_stats) context=kernel
shell@JTY:/ $ su    //切换到su  
shell@JTY:/ #        //出现这个就是具备root了

```

若不具备root权限则可能会提示

```shell
shell@JTY:/ $ su u0_a49
shell@JTY:/ $su
sh: su: can't execute: Permission denied
```



10、总结

编译debug、eng版本一共需要修改以下文件

```shell
（1）frameworks/base/cmds/app_process/app_main.cpp 

（2）frameworks/base/core/jni/com_android_internal_os_Zygote.cpp 

（3）system/core/include/private/android_filesystem_config.h 

（4）system/extras/su/su.c

（5）system/core/init/init.c 
```



编译user版本一共需要修改以下文件

```
（1）frameworks/base/cmds/app_process/app_main.cpp 

（2）frameworks/base/core/jni/com_android_internal_os_Zygote.cpp 

（3）system/core/include/private/android_filesystem_config.h 

（4）system/extras/su/su.c

（5）system/core/init/init.c 

（6）system/extras/su/Android.mk

（7）build/target/product/core.mk
```



