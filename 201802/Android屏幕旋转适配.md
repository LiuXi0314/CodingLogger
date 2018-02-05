# Android屏幕旋转适配

标签（空格分隔）： Android 屏幕旋转

---

###默认条件下屏幕旋转时Activity 的生命周期：

> //销毁
1. onPause
2. onSaveInstanceState
3. onStop
4. onDestroy
//构造
5. onCreate
6. onStart
7. onRestoreInstanceState
8. onResume


###拒绝重新构建Activity

activity 重新构建会造成很多不必要的资源浪费，并且给用户很不良好的交互体验，为了对此种行为说**NO**，我们为当前Activity 在Manifest文件中添加属性配置  

> android:configChanges="orientation|screenSize"


```java
<activity
    android:name=".MainActivity"
    android:configChanges="orientation|screenSize">
</activity>
```
此时Activity的生命周期
```
onConfigurationChanged
```
此时，Activity在屏幕旋转后仅仅触发**onConfigurationChanged()**方法，相对于默认情况省去了7步操作，且不需要重新构建View 和进行数据请求。当然此配置也会导致一系列的问题,如重复的初始化，降低程序效率是必然的了，而且更有可能因为重复的初始化而导致数据的丢失。这是需要千万避免的。。

接下来，我们来了解一下**onConfigurationChanged()**方法，看看此方法是如何为我们省去那多余的七个步骤的
```java
    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
        Logger.d("onConfigurationChanged");
    }
```

```java
/**
 * This class describes all device configuration information that can
 * impact the resources the application retrieves.  This includes both
 * user-specified configuration options (locale list and scaling) as well
 * as device configurations (such as input modes, screen size and screen orientation).
 * <p>You can acquire this object from {@link Resources}, using {@link
 * Resources#getConfiguration}. Thus, from an activity, you can get it by chaining the request
 * with {@link android.app.Activity#getResources}:</p>
 * <pre>Configuration config = getResources().getConfiguration();</pre>
 */
public final class Configuration implements Parcelable, Comparable<Configuration> {
    /*
    *此处省略
    */
}
```

###禁止横竖屏切换
>推荐普通不需要旋屏的app采用

屏蔽横竖屏切换即强制横屏/竖屏
实现方式有以下两种：
1.对AndroidManifest.xml的禁止转向的activity配置中添加属性 
```
android:screenOrientation="landscape/portrait"     
```
2.为项目中所有Activity添加统一父类：BaseActivity，并重写BaseActivity的onCreate()方法，动态设置Activity的横竖屏属性：

```
public class BaseActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);//竖屏
        setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);//横屏     
    }
}
```

