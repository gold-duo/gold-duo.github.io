---
title: 记录1
date: 2021-8-31 23:24
categories: [android]
tags: [android,记录] 
---

## 1.判断是否为debug包

```java
return (context.getApplicationInfo().flags & ApplicationInfo.FLAG_DEBUGGABLE) != 0;
```
## 2.一些io操作常用函数

```java
public static void slientClose(Closeable closeable) {
	if (closeable != null) {
		try {
			closeable.close();
		} catch (IOException e) {
		}
	}
}
private static void copyAssets2FileDir(String filename){
	InputStream inputStream=null;
	OutputStream outputStream=null;
	try {
		inputStream=getAssets().open(filename);
		outputStream=openFileOutput(filename, Context.MODE_PRIVATE);
		int n=0;
		final byte []buffer=new byte[1024];
		while ((n=inputStream.read(buffer))>0){
			outputStream.write(buffer,0,n);
		}
		outputStream.flush();
	} catch (IOException e) {
		e.printStackTrace();
	}finally {
		silentClose(inputStream);
		silentClose(outputStream);
	}
}
public static boolean saveStream2File(InputStream is, File file) {
	FileOutputStream fos = null;
	try {
		fos = new FileOutputStream(file);
		final byte[] buff = new byte[1024];
		int n = 0;
		while ((n = is.read(buff)) > 0) {
			fos.write(buff, 0, n);
		}
		fos.flush();
		return true;
	} catch (FileNotFoundException e) {
	} catch (IOException e) {
	} finally {
		silentClose(fos);
	}
	return false;
}
```
## 3.ndk编译可执行程序

```
Android.mk里include $(BUILD_SHARED_LIBRARY)改成include $(BUILD_EXECUTABLE)
```
## 4.ndk获取到当前进程名

```c
//方法1
extern char *__progname;
//方法2
#include<sys/prctl.h>，prctl(PR_GET_NAME,buffer);限制16个字符
```
## 5.

```gradle
dexOptions {
	preDexLibraries = false
	incremental true
	javaMaxHeapSize "4g"
}
```
## 6.直接拷贝so到lib目录打包不到问题。
- 1.在module目录下建立libs存放jar
- 2.在module目录下src\main建立jniLibs存放so
	
## 7.NDK获取os sdk版本

```c
#include <sys/system_properties.h>
int inline getSdkVer() {
	char buff[10] = {0};
	property_get("ro.build.version.sdk", buff);
	return strlen(buff) > 0 ? atoi(buff) : 0;
}
```
## 8.addr2line用法
- 位置 ndk-bundle\toolchains\<armeabi>\prebuilt\windows-<armeabi>\bin
- 调用参数-C -f -e obj/local/armeabi/<libXX.so> <address> 
- D:\android\sdk\ndk-bundle\toolchains\aarch64-linux-android-4.9\prebuilt\windows-x86_64\bin\aarch64-linux-android-addr2line -f -e   D:\workspace\android_studio\VCSP_pack\AndroidPacker\app\build\libs\arm64-v8a\libkeyinfo.so 00000000000bd438


**00000000000bd438**为输出地址

## 9.编译64位jni "undefined reference to property_get"
	找到一台arm平台的Android 5.1的设备或虚拟机，把/system/lib64/libc.so取出替换NDK里面的库即可。

## 10.执行ndk程序"only position independent executables (PIE) are supported."
> PIE这个安全机制从4.1引入

```
LOCAL_CFLAGS += -pie -fPIE
LOCAL_LDFLAGS += -pie -fPIE
```

## 11. Gradle sync failed: SSL peer shut down incorrectly
> 下载不了gradle.在android studio里指定gradle本地目录

> gradle 使用下载好的本地包

- 编辑 gradle\wrapper\gradle-wrapper.properties 文件
- 将 #distributionUrl=https\://services.gradle.org/distributions/gradle-2.14.1-all.zip
- 改成distributionUrl=file:///F:/gradle-3.3-all.zip
	
## 12.Proguard编译时去除方法调用

```
-assumenosideeffects class android.util.Log {
        public static *** d(...);
        public static *** e(...);
        public static *** i(...);
        public static *** v(...);
        public static *** println(...);
        public static *** w(...);
        public static *** wtf(...);
}
```

## 13.gradle下载不到依赖库
 	repositories{ jcenter()}的jcenter()改为jcenter{url "http://jcenter.bintray.com/"}

## 14.根据屏幕缩放bitmap
- 1.获取bitmap大小

```java
BitmapFactory.Options options = new BitmapFactory.Options();
options.inJustDecodeBounds = true;
BitmapFactory.decodeXX解码图片options.outXX返回图片原始宽高
```
- 2.通过options.outXX计算inSampleSize

```java
private int calcInSampleSize(int width, int targetWidth) {
	int ret = 1;
	while (width >= (targetWidth <<= 1)) {
		ret <<= 1;
	}
	return ret;
}
```
- 3.用BitmapFactory.decodeXX解码


## 15.launchMode
- standard  每次都会新建，每个Task都可以有，且每个Task都可以有多个实例（每个Task都可以有，且可以有多个）  
- singleTop 栈顶复用模式。当前实例如果在栈顶，就不新建实例，调用其OnNewIntent。 如不在栈顶，则新建实例  （每个Task都可以有，且可以有多个，在栈顶时可复用）  
- singleTask 新建一个Task，如果已经有其他的Task并且包含该实例，那就直接调用那个Task的实例。（只有一个Task中会有）  
 - singleInstance 新建一个Task，且在该Task中只有它的唯一一个实例。 (只有一个Task会有，且该Task中只有它)  

    - singleTop,如果不在栈顶则新起activity在栈顶
    - singleTask,如果不在栈顶会弹出前面的activity让自己在栈顶
    - singleInstance,跟singleTask一样但它独立一个栈。（如果此activity被其他activity覆盖，它（独立栈）将在远栈下面）

## 16.安装到system分区

```sh
# 将apk推送到sdcard目录
adb push xxx.apk /sdcard/
# 进入shell
adb shell
# 获取root权限
su
# 使/system分区可读写
mount -o remount /system
# 将xxx.apk装到/system/app中
busybox mv /sdcard/xxx.apk /systerm/app 
# 修改apk文件为root权限(不是必须)
chown root:root /system/app/xxx.apk 
# 将权限设置成和其它系统应用相同(不是必须)
busybox g-w /system/app/xxx.apk
```

## 17.bindService调用stopService不会停止service,所以必须unbind前后台通讯
- 1.AIDL
- 2,broadcastReceiver
- 3,ContentProvider(ContentResolver注册ContentObserver，获取到数据更改通知）
- 4,socket
- 5,MemoryFile

- singleTop,如果不在栈顶则新起activity在栈顶
- singleTask,如果不在栈顶会弹出前面的activity让自己在栈顶
- singleInstance,跟singleTask一样但它独立一个栈。（如果此activity被其他activity覆盖，它（独立栈）将在远栈下面。**singleInstance会导致home键后台点桌面图标回不到之前activity**）

## 18.gradle 排除依赖

```gradle
compile(project(':react-native-custom-module')) {
    exclude group: 'com.facebook.react', module: 'react-native'
}
```
## 19.编译android studio内存不足问题

```toml
[vmoptions]
#-Xmx1280m
#-XX:MaxPermSize=350m
#-d64
-server
-Xms512m
-Xmx2048m
-XX:MaxPermSize=512m
-Xss4m
-ea

-XX:ReservedCodeCacheSize=240m
#-XX:+UseConcMarkSweepGC
-XX:+UseCompressedOops
-XX:+UseCodeCacheFlushing
-XX:+UseConcMarkSweepGC
-XX:MinHeapFreeRatio=15
-Xincgc

-XX:SoftRefLRUPolicyMSPerMB=50
-da
-Djna.nosys=true
-Djna.boot.library.path=

-Djna.debug_load=true
-Djna.debug_load.jna=true
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true
-XX:+HeapDumpOnOutOfMemoryError
-Didea.paths.selector=AndroidStudio2.1
-Didea.platform.prefix=AndroidStudio
[gradle.properties]
org.gradle.jvmargs=-server -Xmx5120m -Dfile.encoding=UTF-8 -Xms512m -XX\:MaxPermSize=512m -Xss4m -ea -XX\:+UseCompressedOops -XX\:+UseCodeCacheFlushing -XX\:+UseConcMarkSweepGC -XX\:MinHeapFreeRatio=15 -Xincgc -XX\:ParallelGCThreads=4
org.gradle.parallel=true
org.gradle.daemon=true
org.gradle.configureondemand=true
android.enableBuildCache=true
```

## 20.加载本地aar
- (1).将aar文件夹(文件夹里有pom)放到项目目录下
- (2).添加maven知道aar文件夹

```gradle
allprojects {
    repositories {
        maven {url "$rootDir/node_modules/react-native/android"}
    }
}
```
## 21.String.format 支持日期格式化

```java
String.format("%1$tY%1$tm%1$td %1$tH:%1$tM:%1$tS", System.currentTimeMillis());
```

## 22.转换hprof
- 生成android.os.Debug.dumpHprofData(hprofPath)
- hprof-conv dump.hprof converted-dump.hprof

## 23.查看进程内存信息
dumpsys meminfo [包名]|[pid]

## 24.依赖冲突
> 在build.gralde忽略掉所有依赖

```
configurations {
    all*.exclude group: 'com.android.support', module: 'support-v4'
}
```
## 25.位操作
- 设置value第几位n为1： value\|=(1<<n)
- 设置value第几位n为0： value&=~(1<<n)
- 判断value第几位n是否为1： value &(1<<n)!=0

## 26.查看model

```sh
cat  /system/build.prop |grep ro.product.model
```
## 27.自绘制控件刷新问题
setWillNotDraw（false）强制必须刷新

## 28.严苛模式 —— StrictMode（在ui线程读写io警告提示）

```java
StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
        .detectDiskReads()
        .detectDiskWrites()
        .detectNetwork()   // or .detectAll() for all detectable problems
        .penaltyLog()
        .build());
StrictMode.VmPolicy.Builder builder = new StrictMode.VmPolicy.Builder();
if (Build.VERSION.SDK_INT >= 18) builder.detectFileUriExposure();
if (Build.VERSION.SDK_INT >= 11) builder.detectLeakedClosableObjects();
if (Build.VERSION.SDK_INT >= 16) builder.detectLeakedRegistrationObjects();
StrictMode.setVmPolicy(builder)
        .penaltyLog()
        .build());
```
## 29.sqlite 将UNIX时间戳转化为时间日期格式

```sql
datetime(timestamp,'unixepoch')
```
## 30.canvas drawText坐标偏移问题
```java
Rect targetRect = new Rect(x, y, w, h);
Paint.FontMetricsInt fontMetrics = paint.getFontMetricsInt();
int baseline = (targetRect.bottom + targetRect.top - fontMetrics.bottom - fontMetrics.top) / 2;
canvas.drawText(text, targetRect.centerX(), baseline, paint);
```
## 31.处理Gradle中的这个文件下载慢的问题
- 1.修改项目根目录下的文件 build.gradle ：

```gradle
buildscript {
    repositories {
        maven{ url 'http://maven.aliyun.com/nexus/content/groups/public/'}
    }
}
allprojects {
    repositories {
        maven{ url 'http://maven.aliyun.com/nexus/content/groups/public/'}
    }
}
```
然后选择重新构建项目就可以了

- 关闭AS,进入`F:\Users\xxx\.gradle\wrapper\dists\gradle-2.14.1-bin\cwymdawcgz00nujebueluurbv`
去gradle网站下载gradle-2.14.1-bin将刚下好的zip文件放到这个目`F:\Users\xxx\.gradle\wrapper\dists\gradle-2.14.1-bin\cwymdawcgz00nujebueluurbv`
重新打开AS即可。

## 32.Cookie设置

```java
cooke是一组 key=value;。。。字符串
CookieSyncManager.createInstance(mContext);
CookieManager cookieManager = CookieManager.getInstance();
cookieManager.setCookie(url,cookieValue);
if (Build.VERSION.SDK_INT < 21) {
    CookieSyncManager.getInstance().sync();
} else {
    CookieManager.getInstance().flush();
}
```
## 33.git提交文件大小写改名
```sh
git config core.ignorecase true
#提交完再
git config core.ignorecase false
```

## 34.打开"Android Device Monitor" 后 "android studio" logcat不显示
设置Tool-->Android-->Enable ADB Integration为勾选状态

## 35.tcpdump抓包
- 下载tcpdump   http://www.strazzere.com/android/tcpdump

 ```sh
 adb push c:\wherever_you_put\tcpdump /data/local/tcpdump
 adb shell chmod 6755 /data/local/tcpdump
 adb shell,   su获得root权限
 cd /data/local
data/local/tcpdump -p -vv -s 0 -w /sdcard/capture.pcap
```

- Typical tcpdump options apply. For example, if you want to see HTTP traffic:  只监听http  

```sh
adb shell tcpdump -X -n -s 0 port 80
```

## 36.中文字体加粗问题

```java
text.getPaint().setStrokeWidth(1);
text.getPaint().setStyle(Paint.Style.FILL_AND_STROKE);
```
## 37.性能优化
- 分析方法调用阻塞 Debug.startMethodTracing/stopMethodTracing
- 分析内存泄漏 Debug.dumpHprofData