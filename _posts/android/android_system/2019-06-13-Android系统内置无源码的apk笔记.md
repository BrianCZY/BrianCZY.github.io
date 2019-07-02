---
layout: post
title: Android系统内置无源码的apk笔记
categories: Android
description: 
keywords: Android, 内置apk，6.0，系统预置应用
---





说明：本次修改的系统源码是基于6.0 系统来修改

参考：https://blog.csdn.net/csdn_pcb/article/details/85867426

1、修改内核，拷贝app到/system/preloadApp/目录下

在vendor目录下创建MyApps目录，在建立对应的项目目录将app拷贝到目录下，比如将FenoMeasure.apk ，HomeWindow.apk和SogouInput.apk内置到系统上，则我所建立的目录结构如下：



```
ubuntu@ubuntu-Pc:~/phone/mtk/kt109/alps/vendor/MyApps$ tree
.
├── FenoMeasure
│   ├── Android.mk
│   └── FenoMeasure.apk
├── HomeWindow
│   ├── Android.mk
│   └── HomeWindow.apk
└── SogouInput
    ├── Android.mk
    └── SogouInput.apk

```



sogouInput/Android.mk   的内容如下：（其他的类似）

```shell
LOCAL_PATH:= $(call my-dir)
 	 include $(CLEAR_VARS)
     LOCAL_MODULE := SogouInput
$(shell mkdir -p $(TARGET_OUT)/preloadApp/$(LOCAL_MODULE)) 
$(shell cp $(LOCAL_PATH)/$(LOCAL_MODULE).apk $(TARGET_OUT)/preloadApp/$(LOCAL_MODULE)/$(LOCAL_MODULE).apk)


```



2、修改系统源码



./frameworks/base/services/core/java/com/android/server/pm/PackageManagerService.java



```java
   if (!mOnlyCore) {
                EventLog.writeEvent(EventLogTags.BOOT_PROGRESS_PMS_DATA_SCAN_START,
                        SystemClock.uptimeMillis());
				//add by brian 2019-6-14---------------
			   //判断为系统第一次启动才做扫描安装操作：
                if(isFirstBoot()){
                    try{
                        installApps();
                    } catch (Exception e) {
                        Log.e(TAG,"Failed to installApps" + e);
                    }
                }


```





```java

private void installApps(){
		//判断vendor/preloadApp 目录是否存在
		File preloadAppDir = new File("/system/preloadApp/");
		if(!preloadAppDir.exists()){
			Log.e(TAG,"/system/preloadApp is not exist");
			return;
		}
		//从vendor/preloadApp中获取apk文件
		String apkDirFilesNames[] = preloadAppDir.list();
		if(apkDirFilesNames == null){
			Log.e(TAG,"apk file name is null");
			return;
		}
		//复制apk文件到 /data/app
		for(int i = 0; i < apkDirFilesNames.length; i++){
			File srcFileDir = new File("/system/preloadApp/", apkDirFilesNames[i]);
			String srcFileNames[] = srcFileDir.list();
			for(int j = 0; j < srcFileNames.length; j++) {
				File srcFile = new File(srcFileDir, srcFileNames[j]);
				File destFileDir =     new File("/data/app/",apkDirFilesNames[i]);
				preparePreloadAppDir(destFileDir);
				boolean installResult = copyPreloadApp(srcFile, destFileDir);
				if(!installResult){
					Log.d(TAG,"install failed");
					return;
				}
			}
		}
	}



	private boolean preparePreloadAppDir(File destFileDir)  {
        //判断destFileDir是否存在
        if (destFileDir.exists()) {
            Log.e(TAG,"dir already exists: " + destFileDir);
            return true;
        }
        try {
            Os.mkdir(destFileDir.getAbsolutePath(), 0755);
            Os.chmod(destFileDir.getAbsolutePath(), 0755);
        } catch (Exception e) {
            // This purposefully throws if directory already exists
            Log.e(TAG,"Failed to prepare session dir: " + destFileDir + e);
            return false;
        }
        if (!SELinux.restorecon(destFileDir)) {
            Log.e(TAG,"Failed to restorecon session dir: " + destFileDir);
            return false;
        }
        return true;
    }

	private boolean copyPreloadApp(File srcFile, File destDir) {
        //检查目录是否存在
        if (srcFile == null || destDir == null || !srcFile.exists()) {
            Log.e(TAG, "invalid arguments");
            return false;
        }
        try{
            // 删除所有.tmp文件
            for (File file : destDir.listFiles()) {
                if (file.getName().endsWith(".tmp")) {
                    file.delete();
                }
            }
            final File tmpFile = File.createTempFile("inherit", ".tmp", destDir);
            if (!FileUtils.copyFile(srcFile, tmpFile)) {
                Log.e(TAG, "Failed to copy " + srcFile + " to " + tmpFile);
                return false;
            }
            try {
                Os.chmod(tmpFile.getAbsolutePath(), 0644);
            } catch (Exception e) {
                Log.e(TAG, "Failed to chmod " + tmpFile);
                return false;
            }
            final File toFile = new File(destDir, srcFile.getName());
            Log.e(TAG, "Renaming " + tmpFile + " to " + toFile);
            if (!tmpFile.renameTo(toFile)) {
                Log.e(TAG, "Failed to rename " + tmpFile + " to " + toFile);
                return false;
            }
        } catch (Exception e) {
            Log.e(TAG, "Failed to copy "  + e);
            return false;
        }
        return true;
    }

```





查看结果

进入adb shell 下，查看/system/preloadApp/ 下是否已有对应的app。

```shell
root@JTY:/ # ls -l /system/preloadApp/
drwxr-xr-x root     root              2019-06-14 21:26 FenoMeasure
drwxr-xr-x root     root              2019-06-14 21:27 HomeWindow
drwxr-xr-x root     root              2019-06-14 20:13 SogouInput

```

