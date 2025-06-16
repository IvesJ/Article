> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/D8yGNUhIFQ8hTtkHkc7eEg)

在 Android 中，当 Activity 的 onCreate 方法中调用 finish() 时，其生命周期会直接跳转至 onDestroy，而不会触发 onStart 和 onResume。具体流程如下：  

生命周期流程
------

1. onCreate()

系统初始化 Activity，开发者在此设置布局和初始化逻辑。当调用 finish() 后，Activity 被标记为 mFinished = true。

 2. onDestroy()

Activity 被销毁，释放资源。系统跳过中间状态（onStart、onResume、onPause、onStop），直接触发 onDestroy。 

源码分析
----

 1. finish() 的调用与标记 •  调用 finish() 会设置 Activity 的 mFinished 标志位，并通知 ActivityManagerService（AMS）销毁该 Activity。 

```
// Activity.java
public void finish() {
    if (mParent == null) {
        ActivityManager.getService().finishActivity(mToken, resultCode, resultData, finishTask);
    } else {
        mParent.finishFromChild(this);
    }
    mFinished = true; // 标记为已结束
}

```

    2. ActivityThread 处理流程 在启动 Activity 时，ActivityThread 依次调用 performLaunchActivity（触发 onCreate）和 handleResumeActivity（触发 onResume）。若 onCreate 中调用了 finish() ： •  performLaunchActivity 调用 onCreate 后，检查 mFinished 标志：

```
// ActivityThread.java
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
    // ...
    if (r.isPersistable()) {
        mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
    } else {
        mInstrumentation.callActivityOnCreate(activity, r.state);
    }
    // 检查 mFinished 标志，若为 true，跳过后续生命周期
    if (!r.activity.mFinished) {
        activity.performStart(); // 触发 onStart
        r.stopped = false;
    }
    // ...
}

```

  由于 mFinished 被标记为 true，performStart() 不会执行，onStart 和 onResume 被跳过。 

 •  handleResumeActivity 被跳过：

ActivityThread 在 handleLaunchActivity 后，若检测到 mFinished 为 true，不再调用 handleResumeActivity。

3. 直接触发 onDestroy 系统通过 ActivityThread 调用 handleDestroyActivity，直接触发 onDestroy：

```
// ActivityThread.java
private void handleDestroyActivity(ActivityClientRecord r, boolean finishing, int configChanges) {
    // ...
    mInstrumentation.callActivityOnDestroy(r.activity);
    // ...
}

```

生命周期验证
------

 通过日志打印验证生命周期调用顺序： 

```
public class TestActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d("Lifecycle", "onCreate");
        finish(); // 调用 finish
    }
    @Override
    protected void onStart() {
        super.onStart();
        Log.d("Lifecycle", "onStart"); // 不会执行
    }
    @Override
    protected void onResume() {
        super.onResume();
        Log.d("Lifecycle", "onResume"); // 不会执行
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        Log.d("Lifecycle", "onDestroy"); // 执行
    }
}

```

  日志输出： 

```
D/Lifecycle: onCreate
D/Lifecycle: onDestroy

```

特殊场景分析
------

 1. finish() 后调用 startActivity 若在 onCreate 中调用 finish() 后立即启动新 Activity： 

```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    finish();
    startActivity(new Intent(this, NextActivity.class));
}

```

  • 生命周期顺序：

onCreate → onDestroy（当前 Activity）→ NextActivity 正常启动。 • 原理：

startActivity 的调用会被加入任务栈，即使当前 Activity 被销毁，系统仍会处理新 Activity 的启动。 

 2. finish() 与 onBackPressed 的区别 • finish() ：直接销毁 Activity，不触发 onBackPressed。 • onBackPressed：默认调用 finish()，但可重写以添加额外逻辑（如确认对话框）。    

总结
--

![](https://mmbiz.qpic.cn/mmbiz_png/yyLvy204xW9Zctycl5vLL5nRqNWmShPaQ3EXgIAxyp1icXJ2nzCtTwCbrYuSx2P6Lp7UU4al7pLkl5d1tcL3RXg/640?wx_fmt=png&from=appmsg)

     通过源码分析可见，onCreate 中调用 finish() 会导致系统跳过中间状态，直接销毁 Activity。这种设计避免了资源浪费，确保未完全初始化的 Activity 不会进入前台。

关注我获取更多知识或者投稿

![](https://mmbiz.qpic.cn/mmbiz_jpg/yyLvy204xW9Uibw4qQxibOBKL1DicLX10o3w57n09uKDowd4ZDjRIgSMUn9cqY6ia77Ys3VfZjG8LUviacGSr0DFIvw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/yyLvy204xW9Uibw4qQxibOBKL1DicLX10o3gibpbVwAGtDUV15FZianjGs1whAZ2gg71IV6J7zQpQhtQRcSyHrGJbxg/640?wx_fmt=jpeg)

```
原文链接：https://juejin.cn/post/7479350849056276532

```