---
layout: post
title: android系统多个usb支持修改笔记
categories: Android
description: android6.0系统修改，支持使用多个usb设备
keywords: Android, usb, 多个usb
---





问题描述：android 6.0的系统插入两个usb设备时只能使用1个usb设备，虽然能够搜索到2个设备但是其中一个

设备的mInterfaces  内容为空。

修改的方法参考：https://www.jianshu.com/p/1d190c43d4a5

1、查找UsbHostManager.java 文件所在地方

```shell

ubuntu@ubuntu-Pc:~/phone/mtk/kt109/alps$ find . -name UsbHostManager.*
./frameworks/base/services/usb/java/com/android/server/usb/UsbHostManager.java
./out/target/common/obj/JAVA_LIBRARIES/services.usb_intermediates/classes/com/an              droid/server/usb/UsbHostManager.class
./out/target/common/obj/JAVA_LIBRARIES/services_intermediates/classes/com/androi              d/server/usb/UsbHostManager.class
ubuntu@ubuntu-Pc:~/phone/mtk/kt109/alps$


```

2、打开UsbHostManager.java 文件进行编辑

```shell
ubuntu@ubuntu-Pc:~/phone/mtk/kt109/alps$ vim ./frameworks/base/services/usb/java/com/android/server/usb/UsbHostManager.java

```



3、在beginUsbDeviceAdded 函数中添加

```java
mNewConfiguration = null;
	mNewInterface = null;
```

修改后的beginUsbDeviceAdded 如下：
```java
 private boolean beginUsbDeviceAdded(String deviceName, int vendorID, int productID,
            int deviceClass, int deviceSubclass, int deviceProtocol,
            String manufacturerName, String productName, String serialNumber) {

        if (DEBUG) {
            Slog.d(TAG, "usb:UsbHostManager.beginUsbDeviceAdded(" + deviceName + ")");
            // Audio Class Codes:
            // Audio: 0x01
            // Audio Subclass Codes:
            // undefined: 0x00
            // audio control: 0x01
            // audio streaming: 0x02
            // midi streaming: 0x03

            // some useful debugging info
            Slog.d(TAG, "usb: nm:" + deviceName + " vnd:" + vendorID + " prd:" + productID + " cls:"
                    + deviceClass + " sub:" + deviceSubclass + " proto:" + deviceProtocol);
        }

        // OK this is non-obvious, but true. One can't tell if the device being attached is even
        // potentially an audio device without parsing the interface descriptors, so punt on any
        // such test until endUsbDeviceAdded() when we have that info.

        if (isBlackListed(deviceName) ||
                isBlackListed(deviceClass, deviceSubclass, deviceProtocol)) {
            return false;
        }

        synchronized (mLock) {
            if (mDevices.get(deviceName) != null) {
                Slog.w(TAG, "device already on mDevices list: " + deviceName);
                return false;
            }

            if (mNewDevice != null) {
                Slog.e(TAG, "mNewDevice is not null in endUsbDeviceAdded");
                return false;
            }

            mNewDevice = new UsbDevice(deviceName, vendorID, productID,
                    deviceClass, deviceSubclass, deviceProtocol,
                    manufacturerName, productName, serialNumber);

            mNewConfigurations = new ArrayList<UsbConfiguration>();
            mNewInterfaces = new ArrayList<UsbInterface>();
            mNewEndpoints = new ArrayList<UsbEndpoint>();
        }

		mNewConfiguration = null;
		mNewInterface = null;
		System.out.println("UsbHostManager  clear ");
        return true;
    }


```



4、重新编译系统

重新编译系统后插入连个usb设备，可见两个usb的mInterfaces内容都显示出来了

```


07-03 15:35:20.809 3706-3706/com.fenobest.fenomeasure_v2 I/System.out: /dev/bus/usb/001/003=UsbDevice[mName=/dev/bus/usb/001/003,mVendorId=17224,mProductId=61697,mClass=239,mSubclass=2,mProtocol=1,mManufacturerName=xx,mProductName=usb,mSerialNumber=1.0,mConfigurations=[
07-03 15:35:20.809 3706-3706/com.fenobest.fenomeasure_v2 I/System.out: UsbConfiguration[mId=1,mName=null,mAttributes=128,mMaxPower=250,mInterfaces=[
07-03 15:35:20.809 3706-3706/com.fenobest.fenomeasure_v2 I/System.out: UsbInterface[mId=0,mAlternateSetting=0,mName=null,mClass=2,mSubclass=2,mProtocol=1,mEndpoints=[
07-03 15:35:20.809 3706-3706/com.fenobest.fenomeasure_v2 I/System.out: UsbEndpoint[mAddress=131,mAttributes=3,mMaxPacketSize=64,mInterval=10]]
07-03 15:35:20.809 3706-3706/com.fenobest.fenomeasure_v2 I/System.out: UsbInterface[mId=1,mAlternateSetting=0,mName=null,mClass=10,mSubclass=0,mProtocol=0,mEndpoints=[
07-03 15:35:20.809 3706-3706/com.fenobest.fenomeasure_v2 I/System.out: UsbEndpoint[mAddress=129,mAttributes=2,mMaxPacketSize=512,mInterval=0]
07-03 15:35:20.809 3706-3706/com.fenobest.fenomeasure_v2 I/System.out: UsbEndpoint[mAddress=1,mAttributes=2,mMaxPacketSize=512,mInterval=0]]]]
07-03 15:35:20.809 3706-3706/com.fenobest.fenomeasure_v2 I/System.out: /dev/bus/usb/001/007=UsbDevice[mName=/dev/bus/usb/001/007,mVendorId=1008,mProductId=42,mClass=0,mSubclass=0,mProtocol=0,mManufacturerName=Hewlett-Packard,mProductName=HP LaserJet Professional P1106,mSerialNumber=null,mConfigurations=[
07-03 15:35:20.809 3706-3706/com.fenobest.fenomeasure_v2 I/System.out: UsbConfiguration[mId=1,mName=null,mAttributes=192,mMaxPower=49,mInterfaces=[
07-03 15:35:20.809 3706-3706/com.fenobest.fenomeasure_v2 I/System.out: UsbInterface[mId=0,mAlternateSetting=0,mName=Printer,mClass=7,mSubclass=1,mProtocol=2,mEndpoints=[
07-03 15:35:20.809 3706-3706/com.fenobest.fenomeasure_v2 I/System.out: UsbEndpoint[mAddress=1,mAttributes=2,mMaxPacketSize=512,mInterval=0]
07-03 15:35:20.809 3706-3706/com.fenobest.fenomeasure_v2 I/System.out: UsbEndpoint[mAddress=129,mAttributes=2,mMaxPacketSize=512,mInterval=0]]

```

