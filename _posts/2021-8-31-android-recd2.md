---
title: 记录2
date: 2021-8-31 23:24
categories: [android]
tags: [android,记录] 
---

## 46.kotlin自动代理

```kotlin
//申明接口
interface IAbcd{
    fun fake()
}
//实现接口
class AbcdImp{
    fun fake(){}
}

//实现代理类
class AbcdDerived(private val d:IAbcd):IAbcd by d{
}

//使用
val a=AbcdImp()
val abcd=AbcdDerived(a)
abcd.fake()
```

## 45.canScrollVertically(x)
- 1.`-1`:是否可向下滚动
- 2.`1`:是否可向上滚动

## 44.一些位运算

```kotlin
fun Int.setBitTrue(pos: Int) = this or (1 shl pos)
fun Int.setBitFalse(pos: Int) = this and ((1 shl pos).inv())
fun Int.getBitBool(pos: Int) = (this and (1 shl pos)) != 0
```

## 43.`unable to find valid certification path to requested target`
- 1.有可能是网络问题。先后重启stuido、pc试试
- 2.上面不行情况下
    - 1.删除android studio包括目录
    - 2.删除
        - 1.\Users\xx\AppData\Local\Google\AndroidStudio*
        - 2.\Users\xx\AppData\Roaming\Google\AndroidStudio*
    - 3.证书存放在
        - 1.AndroidStudio目录下的jre\lib\security

## 42.GestureDetector处理手势

```kotlin
//重载View.dispatchTouchEvent
 override fun dispatchTouchEvent(ev: MotionEvent?): Boolean {
    val oldDragging = dragging
    when (ev.actionMasked) {
        MotionEvent.ACTION_DOWN -> {
            calledFling = false
            parent.requestDisallowInterceptTouchEvent(true)
        }
        MotionEvent.ACTION_MOVE -> {
            parent.requestDisallowInterceptTouchEvent(true)
        }
        MotionEvent.ACTION_CANCEL,MotionEvent.ACTION_UP -> {
            if (oldDragging) {
                if (!calledFling) {
                    //拖动结束
                }
                dragging = false
            }
            parent.requestDisallowInterceptTouchEvent(false)
        }
        else->{
            parent.requestDisallowInterceptTouchEvent(false)
        }
    }
    getGD().onTouchEvent(ev)
    return super.dispatchTouchEvent(ev)
 }
private fun getGD():GestureDetector{
    if (gestureDetector != null) return gestureDetector!!
    val listener= object : GestureDetector.SimpleOnGestureListener() {
        private var totalY = 0F
        private var totalY = 0F
        override fun onScroll(e1: MotionEvent?, e2: MotionEvent?, distanceX: Float,distanceY: Float): Boolean {
            //处理滚动事件(e1为ACTION_DOWN,e2为当前ACTION_MOVE。)。eg:拖动。
            if (!dragging) {
                dragging = abs(distanceX) < abs(distanceY)
                if(dragging){
                    totalY = 0F
                }
            }
            if (dragging) {
                totalY -= distanceY//注意distance要取反！
                //处理拖动事件
            }
            return dragging
        }
        override fun onFling(e1: MotionEvent?,e2: MotionEvent?,velocityX: Float, velocityY: Float): Boolean {
            //动画加速度计算工时:duration=minDuration.coerceAtMost(abs(1000*toatalDistance/velocity).toLong())
            //1000是GestureDetector在使用VelocityTracker计算速度的单位
            dragging = false
            calledFling=true
            //处理快速滚动。
            return true
        }
    }
    gestureDetector = GestureDetector(context, listener)
    return gestureDetector!!
} 
```
## 41.判断touch点是否在某个子View内

```kotlin
    private fun MotionEvent.isPointInView(): Boolean {
        val view=appBarLayout
        if (view != null) {
            val loc=IntArray(2)
            val x = rawX.toInt()//屏幕坐标
            val y = rawY.toInt()//屏幕坐标
            view.getLocationOnScreen(loc)
            val l = loc[0]
            val t = loc[1]
            if (x < l || y < t) {
                return false
            }
            val r = l + view.width
            if (x > r) {
                return false
            }
            val b = t + view.height
            return y <= b
        }
        return false
    }
```

## 40.CoordinarLayout嵌套RecyclerView。CoordinarLayout重写`dispatchTouchEvent`导致touch事件触发ACTION_CANCEL并引发RecyclerView的ITEM触发click事件。

```kotlin
//1.dispatchTouchEvent重载最后一定要回调父类的super.dispatchTouchEvent!!!
//2.RecyclerView的Behavior的onInterceptTouchEvent、onTouchEvent函数需要考虑CoordinarLayout重写`dispatchTouchEvent`是否drag之类的处理则要返回true
```

## 39.`Error while executing process \3.10.2.4988404\bin\ninja.exe with arguments {`
更换低版本ndk比如`android-ndk-r13b`

## 38.代码创建GradientDrawable

```
//放式创建的Drawable必须设置高度（不然有坑）
private Drawable createGradientDrawable(String startColor, String endColor){
    GradientDrawable drawable=null;
    try{
        final int sColor=parseColor(startColor,0xFF27BCEB );
        final int eColor=parseColor(endColor,0xFF42ECF3);
        final int[] colors = {sColor,eColor};
        drawable = new GradientDrawable(GradientDrawable.Orientation.LEFT_RIGHT,colors);
        final int dp=24;
        drawable.setCornerRadius(dp);
        drawable.setGradientType(GradientDrawable.RECTANGLE);
    }catch (Exception e){

    }
    return drawable;
}
```
## 37.RecyclerView.setMaxRecycledViews是指了缓存还导致频繁ViewHolder
检查是否设置了过大的`setItemViewCacheSize`

## 36.TextView设置了`gravity`为`center`但字体仍不居中问题
    设置textAlignment="center"

## 35.PopupWindow坑
- 1.showAtLocation
    - 1.**parent参数并不是anchor**(仅用于获取windowToken)
    - 2.**gravity参数是PopupWindow显示的位置**(跟FrameLaout的gravity一样)
- 2.setAnimationStyle会导致Acitivity之间切换也产生进入/退出动画。把动画设置到contentView(dismis时设置退出动画)
- 3.在构造函数里必须传入宽高，否则可能出现show显示不出来的问题

## 34.RecyclerView StaggeredGridLayoutManager占满整行

```
//在holder的itemView中
ViewGroup.LayoutParams lp=itemView.getLayoutParams();
if(lp instanceof StaggeredGridLayoutManager.LayoutParams){
    StaggeredGridLayoutManager.LayoutParams lp2=(StaggeredGridLayoutManager.LayoutParams)lp;
    lp2.setFullSpan(true);//占满整行
    lp2.width=StaggeredGridLayoutManager.LayoutParams.MATCH_PARENT;
    itemView.setLayoutParams(lp);
}
```

## 33.通过gradle脚本给`BuildConfig.java`添加变量变量

```java
android {
    boolean arg=project.hasProperty("testCoverage") && Boolean.valueOf(project.testCoverage)
    buildTypes {
        release {
            buildConfigField "boolean", "myTest", arg.toString()
        }
        debug {
            buildConfigField "boolean", "myTest", arg.toString()
        }
    }
}
//上面的代码就会在BuildConfig类中增加boolean变量myTest
```

## 32. No toolchains found in the NDK toolchains folder for ABI with prefix: mips64el-linux-android
问题原因时ndk删除了mips64el-linux-android
1.下载旧版ndk

```
https://developer.android.google.cn/ndk/downloads/older_releases?hl=zh_cn
下载旧版ndk 	android-ndk-r12b
```
2.在`local.properties`中设置`ndk.dir=D\:\\D_Driver\\android\\sdk\\ndk\\android-ndk-r12b`

## 31.adb wifi链接
- 1.获取ip地址
	- 1.adb -s db8ec550  shell ip addre show wlan0
	- 2..adb -s db8ec550  shell ip addre show rmnet0

- 2.链接
adb connect ip:port
adb connect 10.80.150.191:5555

## 31.Fragment嵌套`FragmentManager`用`getChildFragmentManager`

## 30.git回滚到某个点
- 1.git reset --hard gitcode    
- 2.git push -f

## 29.软键盘弹出导致PopupWindow上移问题
mainfest文件中添加：
`android:windowSoftInputMode="adjustNothing"`

## 28.`Installation failed due to: 'device offline'`
- 1.clean整个项目
- 2.重启电脑
- 3.编译运行
- 4.不行重复上面3个步骤

## 27.Build时`Build Output乱码`
双击shift，输入“Edit Custom VM Options”点击添加下面一行` -Dfile.encoding=UTF-8`重启

## 26.oppo安裝測試包`The application could not be installed: INSTALL_FAILED_TEST_ONLY `
gradle.properties文件末尾添加一句：android.injected.testOnly=false

## 25.着不到"support.v7"问题：gradle.properties

```ini
# AndroidX package structure to make it clearer which packages are bundled with the
# Android operating system, and which are packaged with your app's APK
# https://developer.android.com/topic/libraries/support-library/androidx-rn
android.useAndroidX=false
# Automatically convert third-party libraries to use AndroidX
android.enableJetifier=false
```

## 24.OnLayoutChangeListener.onLayoutChange 用setLayoutParams更新视图大小失效问题
    用postDelay运行用setLayoutParams更新视图大小失效问题


## 23.获取系统activity切换动画

```java
private Animation loadDefaultAnim() {
    Resources.Theme theme = mActivity.getTheme();
    if (theme == null) {
        return null;
    }
    int rv;
    TypedArray ta = null;
    try {
        ta = theme.obtainStyledAttributes(android.R.style.Animation_Activity, new int[]{android.R.attr.activityOpenEnterAnimation/*,android.R.attr.activityOpenExitAnimation*/});
        if (ta == null) {
            return null;
        }
        rv = ta.getResourceId(0, 0);
        if (rv != 0) {
            Animation anim = AnimationUtils.loadAnimation(mActivity, rv);
            return anim;
        }
    } catch (Exception e) {
    } finally {
        if (ta != null) {
            ta.recycle();
        }
    }
    return null;
}
```

## 22.沉浸式状态栏
- 1.fitsSystemWindows会导致系统在顶部插入状态栏等高的间距。
    可以通过设置content容器的WindowInsets清除掉系统插入的状态栏等高间距。

```java
final View.OnApplyWindowInsetsListener onApplyWindowInsetsListener = new View.OnApplyWindowInsetsListener() {
    @Override
    public WindowInsets onApplyWindowInsets(View v, WindowInsets insets) {
        if (insets == null || !insets.hasSystemWindowInsets()) return insets;
        return insets.replaceSystemWindowInsets(insets.getSystemWindowInsetLeft(), 0, insets.getSystemWindowInsetRight(), 0);
    }
};
findViewById(android.R.id.content).setOnApplyWindowInsetsListener(onApplyWindowInsetsListener);
```
- 2.AppBarLayout中CollapsingToolbarLayout底下也可以加入view,而这个view随着上移可以用作置顶。
- 3.Toolbar是一个ViewGroup里面可以放子view

## 21.adb forward实现pc和app通信
- 1.adb forward tcp:pc端口 tcp:app端口
- 2.建立socket server和socket client

## 20.ViewTreeObserver.addOnPreDrawListener导致Layout Inspector工具捕获不到师徒

## 19.RecyclerView的header显示在CoordinatorLayout 的AppBarLayout上的错乱问题
- 1.将RecyclerView的位置移动到AppBarLayout前面
- 2.不要代码去动态调整RecyclerView、AppBarLayout的TranslationY设置Y坐标。
- 2.CoordinatorLayout设置fitsSystemWindows也有可能导致RecyclerView位移错乱

## 18.fitsSystemWindows 
- 1.只有将statusbar设为透明，或者界面设为全屏显示（设置View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN flag)时，fitsSystemWindows才会起作用。
- 2.如果多个view同时设置了fitsSystemWindows，**只有第一个会起作用**。
- CoordinatorLayout对fitsSystemWindows主要做了以下处理。 
    - 1.将界面设为全屏。view.setSystemUiVisibility(View.SYSTEM_UI_FLAG_LAYOUT_STABLE\| View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN); 
    - 2.自己绘制statusbar背景。setStatusBarBackground函数可以设置statusbar背景。或者在布局文件中通过app:statusBarBackground设置。 
    - 3.如果CoordinatorLayout的子View没有设置fitsSystemWindows，在layout时将子Viwe向下偏移statusbar的高度，用来显示CoordinatorLayout绘制的statusbar。如果子view设置了fitsSystemWindows，子View会覆盖CoordinatorLayout的statusbar。setStatusBarBackground设置的状态栏 
    将被覆盖，不再起作用。具体逻辑可参考CoordinatorLayout的layoutChild 函数。 
    - 4.调用dispatchApplyWindowInsets，让子view的behavior或子view接着处理fitsSystemWindows属性。CoordinatorLayout的很多常用的子view如AppBarLayout也对fitsSystemWindows进行了个性化处理。

## 17.`clipChildren`是否限制子View在父View范围内（false不限制）

 
## 16.adb 指定作用设备
    adb -s  HC359W909946
    
## 15.使用CoordinatorLayout 抛"You need to use a Theme.AppCompat theme (or descendant) with the design library."异常
 设置ThemeTheme.AppCompat

## 14.shape模板
- 1.drawable放在drawable文件夹

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <stroke android:width="0.5dp" android:color="@color/vip_red" />
    <corners android:radius="20dp" />
    <solid android:color="@color/white" />
</shape>
```

## 13.selector模板
- 1.drawable放在drawable文件夹

```xml
<?xml version="1.0" encoding="UTF-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@android:color/transparent" android:state_checked="false"></item>
    <item android:drawable="@drawable/all_newest_pro_bottom_checked" android:state_checked="true"></item>
</selector>
```
- 2.textColor 放在color文件夹

```xml
<?xml version="1.0" encoding="UTF-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:color="#000000" android:state_checked="false"></item>
    <item android:color="#E4007F" android:state_checked="true"></item>
</selector>
```
## 12.计算TextView文本占用行数

```java
   private Layout createLayout(@NonNull TextView textView,@Nullable CharSequence text) {
        if (TextUtils.isEmpty(text)) {
            return null;
        }
        int text_name_width = getStoryTextViewWidth();
        Layout sl;
        if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.P) {
            DynamicLayout.Builder builder = DynamicLayout.Builder.obtain(
                    text,
                    textView.getPaint(),
                    text_name_width
            );
            sl = builder
                    .setAlignment(Layout.Alignment.ALIGN_NORMAL)
                    .setLineSpacing(0f,0f)
                    .setIncludePad(false)
                    .setBreakStrategy(textView.getBreakStrategy())
                    .setUseLineSpacingFromFallbacks(textView.isFallbackLineSpacing())
                    .setHyphenationFrequency(textView.getHyphenationFrequency())
                    .setJustificationMode(textView.getJustificationMode())
                    .setEllipsize(textView.getEllipsize())
                    .build();
        }else if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.M) {
            StaticLayout.Builder builder = StaticLayout.Builder.obtain(
                    text,
                    0,
                    text.length(),
                    textView.getPaint(),
                    text_name_width
            );
            sl = builder
                    .setAlignment(Layout.Alignment.ALIGN_NORMAL)
                    .setLineSpacing(0f,0f)
                    .setIncludePad(false)
                    .setBreakStrategy(textView.getBreakStrategy())
                    .setHyphenationFrequency(textView.getHyphenationFrequency())
                    .setEllipsize(textView.getEllipsize())
                    .build();
        } else {
            sl = new StaticLayout(
                    text,
                    textView.getPaint(),
                    text_name_width,
                    Layout.Alignment.ALIGN_NORMAL,
                    0f,
                    0f,
                    false
            );
        }
        MyLog.info(getClass(), sl.getLineCount() + "--" + text);
        return sl;
    }
```

## 11.检查字符是否中文

```java
    private static boolean isChinese(char c) {
        Character.UnicodeBlock ub = Character.UnicodeBlock.of(c);
        return ub == Character.UnicodeBlock.CJK_UNIFIED_IDEOGRAPHS
                || ub == Character.UnicodeBlock.CJK_COMPATIBILITY_IDEOGRAPHS
                || ub == Character.UnicodeBlock.CJK_UNIFIED_IDEOGRAPHS_EXTENSION_A
                || ub == Character.UnicodeBlock.CJK_UNIFIED_IDEOGRAPHS_EXTENSION_B
                || ub == Character.UnicodeBlock.CJK_SYMBOLS_AND_PUNCTUATION
                || ub == Character.UnicodeBlock.HALFWIDTH_AND_FULLWIDTH_FORMS
                || ub == Character.UnicodeBlock.GENERAL_PUNCTUATION;
    }
```

## 10.设置代理后导致gradle下载问题
查看在c:\Users\xxuser\.gradle目录下文件gradle.properties是否设置了代理

## 9.监控界面渲染耗时

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
	getWindow().addOnFrameMetricsAvailableListener(new Window.OnFrameMetricsAvailableListener() {
		List<Integer> frameDurations = new ArrayList<>(100);
		@Override
		public void onFrameMetricsAvailable(Window window, FrameMetrics frameMetrics, int dropCountSinceLastInvocation) {
			frameDurations.add((int) (frameMetrics.getMetric(TOTAL_DURATION) / 1000000));
			if (frameDurations.size() == 100) {
				getWindow().removeOnFrameMetricsAvailableListener(this);
				L.w("AllCategory", Arrays.toString(frameDurations.toArray()));
			}
		}
	}, new Handler(Looper.getMainLooper()));
}
```

## 8.从堆栈获取类名、方法名、调用方法名、调用类名

```java
StackTraceElement[] stacktrace = Thread.currentThread().getStackTrace();

//当前方法的"类名.方法名()"
StackTraceElement element = stacktrace[2];
element.getClassName() + '.' + element.getMethodName() + "()";

//调用者的"类名.方法名()"
StackTraceElement element = stacktrace[3];
element.getClassName() + '.' + element.getMethodName() + "()";
```
## 7.sqlite忽略大小写

```sql
SELECT * FROM User WHERE UserName = ‘user1′ COLLATE NOCASE;
```


## 6. 获取apk证书指纹

```
命令keytool -list -v -keystore <keystore-file>
```

## 5. “/data/local/tmp” remote Permission denied

```sh
# cd /data/local
# su
# mv tmp tmp-old # (or simply rm -rf tmp)
# mkdir /mnt/sdcard/tmp
# ln -s /mnt/sdcard/tmp ./tmp
```

## 4. asm 注入代码android 编译出现"Uncaught translation error: com.android.dx.cf.code.SimException: mismatched stack depths"错误问题

```kotlin
val classWriter = ClassWriter(classReader, ClassWriter.COMPUTE_MAXS);
//classReader参数不能忽略一定要传
```

## 3. 监听截屏
核心的思路是监听相册里有没有新文件生成按截图的名称规则和生成时间来匹配

```
ContentObserver 方法：注册 MediaStore.Images.Media.EXTERNAL_CONTENT_URI，onChanage时从MediaStore.Images.Media.EXTERNAL_CONTENT_URI ContentResolver 便利Cursor通过名称生成时间两个条件来判断是否刚生成的截图
```

##  2. Activity三级跳无法传回返回值给第一个Activity问题

```java
startActivityForResult时在 intent.addFlags(Intent.FLAG_ACTIVITY_FORWARD_RESULT);
```
> FLAG_ACTIVITY_FORWARD_RESULT：

如果给Intent对象设置了这个标记，并且这个Intent对象被用于从一个既存的Activity中启动一个新的Activity，然后将这个既存Activity的回复目标转移到新的Activity。使用这种方式获取的新的Activity能够调用setResult(int)方法，把结果返回给原始的Activity。

```java
    private static final String[] KEYWORDS = {
            "screenshot", "screen_shot", "screen-shot", "screen shot",
            "screencapture", "screen_capture", "screen-capture", "screen capture",
            "screencap", "screen_cap", "screen-cap", "screen cap"
    };
```

##  1. launchMode="singleTask" 唤起微信/QQ返回不了当前页面问题
- singleTask 新建一个Task，如果已经有其他的Task并且包含该实例，那就直接调用那个Task的实例。（只有一个Task中会有）  
- 微信/QQ返回APP默认栈，所以返回不到换回页面。
解决方法：用中转activity（launchMode="standard"）来唤起微信/QQ,唤起后该activity立即关闭，
activity theme设置为：

```xml
<style name="rn_transparent_theme" parent="@android:style/Theme.Dialog">
    <item name="android:windowIsTranslucent">true</item>
    <item name="android:windowFrame">@android:color/transparent</item>
   <!-- <item name="android:windowContentOverlay">@null</item>-->
    <item name="android:windowNoTitle">true</item>
    <item name="android:windowTitleStyle">@null</item>
    <item name="android:windowAnimationStyle">@null</item>
    <item name="android:windowCloseOnTouchOutside">false</item>
    <item name="android:windowBackground">@android:color/transparent</item>
</style>
```

