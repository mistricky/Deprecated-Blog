# ADB 连接不上 Android 设备

博主的机器是 mac，在初次连接 Android 设备的时候发现连接不上。

这里分享一下解决办法

出现这种情况主要是因为adb内建有一个知名的厂商ID列表，对于列表内的设备，adb可以直接连接，而不在列表中的设备，则不好意思，它会直接返回，这也就是为什么android设备的驱动已经安装好了，而adb连接不上的原因

那 adb 靠什么来辨识呢？就是厂商ID，也就是我们平时说的 vendorID，如果你是 mac， 你可以用

```
system_profiler SPUSBDataType
```

来查看设备的 vendorID

![在这里插入图片描述](https://img-blog.csdn.net/20181021233011604?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0hhb0RhV2FuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

比如这里是我的设备的相关信息

接下来可以去 /Users/[user]/.android 目录下去看一下有没有`adb_usb.ini`这个文件。如果没有可以新建一个，然后再在里面添加设备的 vendorID

```
0x18d1
```

然后接下来就可以正常连接了！

使用

```
adb devices -l
```

进行查看

如果还是不行，请查看一下`usb调试`是否开启

