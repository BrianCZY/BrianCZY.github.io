---


layout: post
title: RecyclerView设置分割线记录
categories: Android
description: Android USB host 模式通讯
keywords: Android, USB, usb,host
---
背景：华为手机9.0，实现USB通讯功能，通过外接转USB线来连接手机typeC接口

Host模式下的通讯概况：

![image-20200526111959107](https://raw.githubusercontent.com/BrianCZY/BrianCZY.github.io/master/images/blog/usb/image-20200526111959107.png)

1、注册USB监听BroadcastReceiver


```kotlin
registerReceiver(context)
```

```kotlin
    val usbReceiver = object : BroadcastReceiver() {

        override fun onReceive(context: Context, intent: Intent) {


            if (intent.action == UsbManager.ACTION_USB_DEVICE_ATTACHED) {
                val device: UsbDevice = intent.getParcelableExtra(UsbManager.EXTRA_DEVICE)
                Log.d("usb", device.deviceName.toString() + "USB插入 " + "VendorId=" + String.format("0x%04x", device.vendorId) + " " + "ProductId=" + String.format("0x%04x", device.productId))
                if (isB1Device(device)) {
                    mDevice = device
                    mContext?.let { requirePremission(it, device) }
                }
            }
            if (intent.action == UsbManager.ACTION_USB_DEVICE_DETACHED) {
                val device: UsbDevice = intent.getParcelableExtra(UsbManager.EXTRA_DEVICE)
                Log.d("usb", "USB拨出 " + "VendorId=" + String.format("0x%04x", device.vendorId) + " " + "ProductId=" + String.format("0x%04x", device.productId))
                if (isB1Device(device)) {
                    mIsConnnected = false
                    close()
                }

            }
            if (intent.action == UsbManager.EXTRA_PERMISSION_GRANTED) {
                val device = intent.getParcelableExtra<UsbDevice>(UsbManager.EXTRA_DEVICE)

                if (intent.getBooleanExtra(UsbManager.EXTRA_PERMISSION_GRANTED, false)) {

                    Log.d("usb", "USB权限申请成功")
                    //TODO Open
//打开设备
                    //打开设备
                    if (isB1Device(device)) {
//                        mConnection = mUsbManager!!.openDevice(device)
                        openDeviceData(device)
                    }

                } else {
                    Toast.makeText(mContext, "USB权限被拒绝", Toast.LENGTH_SHORT).show()
                    Log.d("usb", "USB权限被拒绝")

                }
            }
        }
    }

    fun registerReceiver(context: Context?) {
        val filter = IntentFilter()
        filter.addAction("android.hardware.usb.action.USB_DEVICE_ATTACHED")
        filter.addAction("android.hardware.usb.action.USB_DEVICE_DETACHED")
        filter.addAction("permission")
        context!!.registerReceiver(usbReceiver, filter)
    }


    fun unRegisterReceiver(context: Context?) {
        context!!.unregisterReceiver(usbReceiver)
    }

```







2、搜索设备

```kotlin
mUsbManager = context.getSystemService(Context.USB_SERVICE) as UsbManager?
```

```kotlin
fun findDevice(context: Context): UsbDevice? {
    val deviceList = mUsbManager!!.deviceList
    deviceList.values.forEach {
        Log.d(TAG, "deviceId : ${it.deviceId}  productId : ${it.productId}   vendorId : ${it.vendorId}")
        if (PRODUCT_ID == it.productId && VENDOR_ID == it.vendorId) return it
    }
    return null
}
```



3、申请权限

```kotlin
mDevice?.run {
    requirePremission(context, this)
}
```

```kotlin
    fun requirePremission(context: Context, device: UsbDevice) {
//        val mUsbManager = context.getSystemService(Context.USB_SERVICE) as UsbManager?
        mUsbManager?.run {
            val mPermissionIntent = PendingIntent.getBroadcast(context, 0, Intent(UsbManager.EXTRA_PERMISSION_GRANTED), 0)
            if (mUsbManager!!.hasPermission(device)) {
                //进行通信相关的操作

                openDeviceData(device)
            } else {
                mUsbManager!!.requestPermission(device, mPermissionIntent);
            }
        }
    }
```



4、打开设备并监听端口数据

打开设备后，开启端口数据监测

```kotlin
//打开设备，获取 读数据端点，写数据端点
private fun openDeviceData(device: UsbDevice) {
    openUsbDevice(device)
    mUsbEpIn?.let {
        receiveUsbData(it)
    }

}
```

打开设备，获取输入&输出端口

```kotlin
private fun openUsbDevice(device: UsbDevice) {
    mConnection = mUsbManager!!.openDevice(device)

    device?.getInterface(0)?.also { intf ->

        mConnection?.claimInterface(intf, true)
        intf.getEndpoint(0)?.also { endpoint ->
            mUsbEpIn = endpoint
        }
        intf.getEndpoint(1)?.also { endpoint ->
            mUsbEpOut = endpoint
        }
    }


}
```

开启协程接收端口的数据，注意这里的数据是在IO协程上，若要将数据显示到界面需要切换到主线程上，如withContext(Dispatchers.Main) 

```kotlin
    //接收数据
    private fun receiveUsbData(usbEpIn: UsbEndpoint) {
        val epMax = usbEpIn!!.maxPacketSize
        mReceiveJob = GlobalScope.launch(Dispatchers.IO) {
            while (true) {

                mConnection?.run {
                    val byteArray = ByteArray(epMax) //byteArray即为读取到的数据
                    val size = mConnection!!.bulkTransfer(usbEpIn, byteArray, byteArray.size, mTimeoutMillis)
//                    Log.i(TAG, "byreArray:$byteArray")
                    if (size > 0) {
                        Log.d(TAG, "size:$size    receive byteArray:${Arrays.toString(byteArray)}")
                        //TODO 解析数据
                    }

                }

                delay(2)
            }
        }


    }
```



5、以上的整体代码如下：

```
package com.brian.manager.usb

import android.app.PendingIntent
import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.content.IntentFilter
import android.hardware.usb.UsbDevice
import android.hardware.usb.UsbDeviceConnection
import android.hardware.usb.UsbEndpoint
import android.hardware.usb.UsbManager
import android.util.Log
import android.widget.Toast
import kotlinx.coroutines.*
import java.util.*
import kotlin.math.min


/**
 * @Desc:
 * Created by brian on 2020/5/25
 */


object MyUsbManager {

    private lateinit var mReceiveJob: Job
    val TAG = MyUsbManager::class.java.simpleName
    //设备硬件信息
    const val PRODUCT_ID = 41221 //41221   0xA105
    const val VENDOR_ID = 1155  //1155    0x438
    private const val mTimeoutMillis = 1000
    private var mConnection: UsbDeviceConnection? = null
    var mUsbManager: UsbManager? = null
    var mContext: Context? = null
    var mDevice: UsbDevice? = null
    var mIsConnnected: Boolean = false

    var mUsbEpIn: UsbEndpoint? = null
    var mUsbEpOut: UsbEndpoint? = null


    fun init(context: Context) {
        mContext = context
        mUsbManager = context.getSystemService(Context.USB_SERVICE) as UsbManager?
        registerReceiver(context)
        mDevice = findDevice(context)
        mDevice?.run {
            requirePremission(context, this)
        }
    }

    fun isB1Device(device: UsbDevice): Boolean {
        return device.productId == PRODUCT_ID && device.vendorId == VENDOR_ID
    }

    fun relaese() {
        unRegisterReceiver(mContext)
        close()
    }

    fun findDevice(context: Context): UsbDevice? {
//        val mUsbManager = context.getSystemService(Context.USB_SERVICE) as UsbManager?
        val deviceList = mUsbManager!!.deviceList
        deviceList.values.forEach {
            Log.d(TAG, "deviceId : ${it.deviceId}  productId : ${it.productId}   vendorId : ${it.vendorId}")
            if (PRODUCT_ID == it.productId && VENDOR_ID == it.vendorId) return it
        }
        return null
    }


    fun requirePremission(context: Context, device: UsbDevice) {
//        val mUsbManager = context.getSystemService(Context.USB_SERVICE) as UsbManager?
        mUsbManager?.run {
            val mPermissionIntent = PendingIntent.getBroadcast(context, 0, Intent(UsbManager.EXTRA_PERMISSION_GRANTED), 0)
            if (mUsbManager!!.hasPermission(device)) {
                //进行通信相关的操作

                openDeviceData(device)
            } else {
                mUsbManager!!.requestPermission(device, mPermissionIntent);
            }
        }
    }

    //打开设备，获取 读数据端点，写数据端点
    private fun openDeviceData(device: UsbDevice) {
        openUsbDevice(device)
        mUsbEpIn?.let {
            receiveUsbData(it)
        }

    }

    //接收数据
    private fun receiveUsbData(usbEpIn: UsbEndpoint) {
        val epMax = usbEpIn!!.maxPacketSize
        mReceiveJob = GlobalScope.launch(Dispatchers.IO) {
            while (true) {

                mConnection?.run {
                    val byteArray = ByteArray(epMax) //byteArray即为读取到的数据
                    val size = mConnection!!.bulkTransfer(usbEpIn, byteArray, byteArray.size, mTimeoutMillis)
//                    Log.i(TAG, "byreArray:$byteArray")
                    if (size > 0) {
                        Log.d(TAG, "size:$size    receive byteArray:${Arrays.toString(byteArray)}")
                        //TODO 解析数据
                    }

                }

                delay(2)
            }
        }


    }


    fun sendUsbData(byteArray: ByteArray): Int {

        if (mUsbEpOut == null) {
            return -1
        }

        val epMax = mUsbEpOut!!.maxPacketSize
        var size = 0
        var sizeLeft = byteArray.size
        var sizeWrite = 0
        mConnection?.run {
            do {
                val array = byteArray.copyOfRange(sizeWrite, sizeWrite + min(sizeLeft, epMax))
                size = mConnection!!.bulkTransfer(mUsbEpOut, array, array.size, mTimeoutMillis)
                if (size <= 0) {
                    Log.d("usb write -> ", "error $size")
                    return -1
                    break
                } else {
                }

                sizeLeft -= size
                sizeWrite += size
            } while (sizeLeft > 0)

        }
        return sizeWrite
    }

    private fun openUsbDevice(device: UsbDevice) {
        mConnection = mUsbManager!!.openDevice(device)

        device?.getInterface(0)?.also { intf ->

            mConnection?.claimInterface(intf, true)
            intf.getEndpoint(0)?.also { endpoint ->
                mUsbEpIn = endpoint
            }
            intf.getEndpoint(1)?.also { endpoint ->
                mUsbEpOut = endpoint
            }
        }


    }

    fun close(): Boolean {
        for (i in 0 until mDevice!!.interfaceCount) {
            mConnection!!.releaseInterface(mDevice!!.getInterface(i))
        }
        mConnection!!.close()
        mReceiveJob.cancel()
        return true
    }


    val usbReceiver = object : BroadcastReceiver() {

        override fun onReceive(context: Context, intent: Intent) {


            if (intent.action == UsbManager.ACTION_USB_DEVICE_ATTACHED) {
                val device: UsbDevice = intent.getParcelableExtra(UsbManager.EXTRA_DEVICE)
                Log.d("usb", device.deviceName.toString() + "USB插入 " + "VendorId=" + String.format("0x%04x", device.vendorId) + " " + "ProductId=" + String.format("0x%04x", device.productId))
                if (isB1Device(device)) {
                    mDevice = device
                    mContext?.let { requirePremission(it, device) }
                }
            }
            if (intent.action == UsbManager.ACTION_USB_DEVICE_DETACHED) {
                val device: UsbDevice = intent.getParcelableExtra(UsbManager.EXTRA_DEVICE)
                Log.d("usb", "USB拨出 " + "VendorId=" + String.format("0x%04x", device.vendorId) + " " + "ProductId=" + String.format("0x%04x", device.productId))
                if (isB1Device(device)) {
                    mIsConnnected = false
                    close()
                }

            }
            if (intent.action == UsbManager.EXTRA_PERMISSION_GRANTED) {
                val device = intent.getParcelableExtra<UsbDevice>(UsbManager.EXTRA_DEVICE)

                if (intent.getBooleanExtra(UsbManager.EXTRA_PERMISSION_GRANTED, false)) {

                    Log.d("usb", "USB权限申请成功")
                    //TODO Open
//打开设备
                    //打开设备
                    if (isB1Device(device)) {
//                        mConnection = mUsbManager!!.openDevice(device)
                        openDeviceData(device)
                    }

                } else {
                    Toast.makeText(mContext, "USB权限被拒绝", Toast.LENGTH_SHORT).show()
                    Log.d("usb", "USB权限被拒绝")

                }
            }
        }
    }

    fun registerReceiver(context: Context?) {
        val filter = IntentFilter()
        filter.addAction("android.hardware.usb.action.USB_DEVICE_ATTACHED")
        filter.addAction("android.hardware.usb.action.USB_DEVICE_DETACHED")
        filter.addAction("permission")
        context!!.registerReceiver(usbReceiver, filter)
    }

    fun unRegisterReceiver(context: Context?) {
        context!!.unregisterReceiver(usbReceiver)
    }


}
```



6、使用：

（1）程序启动后需要初始化管理类 MyUsbManager.init()

（2）发送数据MyUsbManager.sendUsbData(bytes) ，这里还可以封装成 onSuccess，onFailure 。具体根据业务来定

（3）接收数据在receiveUsbData方法中，byteArray即为接收到的数据，根据具体业务可以进一步封装数据解析方法。

