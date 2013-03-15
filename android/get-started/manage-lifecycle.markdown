#启动一个 android.app.Activity

与普通程序使用 main() 函数作为入口函数不同，android 系统根据系统当前的 lifecycle 状态，通过调用不同的 callback methods 来启动 android.app.Activity

##理解 android.app.Activity

![Alt lifecycle](images/basic-lifecycle.png "android lifecycle")

系统的 lifecycle 就像一个金字塔，当系统创建 android.app.Activity 时，系统的 lifecycle 一步一步上升；当用户离开 android.app.Activity 的时候，系统的 lifecycle 一步一步下降。系统的 lifecycle 主要有如下阶段：

- Resumed/Running: 用户与 android.app.Activity 交互的状态  
- Paused: android.app.Activity 被其他 android.app.Activity 部分遮挡，处于 Puased 状态的 android.app.Activity 不能接收用户的任何输入，也不能继续执行代码  
- Stopped: 用户完全看不到该 android.app.Activity。当处于 Stopped 状态的时候，android.app.Activity 的 instance 以及它的状态信息都会被保存，但是也不能继续执行任何代码  
- Created/Started: 这两种状态只是一种暂时的过渡状态  

##指定启动 android.app.Activity

默认情况下，系统认为 MainActivity() 是一个 android.app.Activity 的主 android.app.Activity，这是可以改变的。只要在声明 android.app.Activity 的时候加上含有 MAIN action 和 LAUNCHER 目录的 <intent-filter> 标签

	<activity android:name=".MainActivity" android:label="@string/app_name">
		<intent-filter>
			<action android:name="android.intent.action.MAIN" />
			<category android:name="android.intent.category.LAUNCHER" />
		</intent-filter>
	</activity>

##创建一个 Instance

系统在创建一个 android.app.Activity 的 Instance 的时候，总会调用 android.app.Activity.onCreate() method。然后系统会调用 android.app.Activity.onStart() 以及 android.app.Activity.onResume() 两个 methods

##销毁 android.app.Activity

一个 android.app.Activity 最先调用的 method 是 android.app.Activity.onCreate()，最后调用的 method 是 android.app.Activity.onDestroy()

#Pausing 和 Resuming 一个 android.app.Activity

##Pause 一个 android.app.Activity

当一个系统调用 android.app.Activity.onPase() 时，开发者一般需要做如下工作

- 停止界面动画等消耗 CPU 的动作  
- 保存一些数据  
- 释放系统资源，比如 broadcast receivers  

##Resume 一个 android.app.Activity

每当 android.app.Activity 出现或创建的时候，系统都会调用 android.app.Activity.onResume() method。开发者可能需要在 android.app.Activity.onResume() method 中，重新创建被 android.app.Activity.onPause() method 释放的资源。

#停止和重新启动一个 android.app.Activity

当一个 android.app.Activity  的 android.app.Activity.onStop() method 被调用的时候，它应该释放大部分资源。当处于 Stopped 状态时，系统在需要释放内存的时候，可能会不通过调用 android.app.Activity.onDestroy() method 而直接销毁这个 android.app.Activity。所以 android.app.Activity.onStop() 里应该释放所有可能引发内存泄漏的资源占用。

尽管系统一般会先调用 android.app.Activity.onPause() method，然后再调用 android.app.Activity.onStop() method，开发者应该在 android.app.Activity.onStop() 里执行消耗时间更长、更消耗 CPU 的操作

##启动/重新启动一个 android.app.Activity

当一个 android.app.Activity 重新启动的时候，系统都会调用 android.app.Activity.onRestart() method，然后会调用 android.app.Activity.onStart() method。前一种方法只会在 android.app.Activity 从 Stopped/Paused 状态切换到 Resumed/Running 状态时才调用的，后者是每当 android.app.Activity 变得可视的时候都会调用的。

一般可以把 android.app.Activity.onStart() method 和 android.app.Activity.onStop() method 做相反的事，前者检查、创建系统资源，后者释放、销毁资源。

有时候，也需要在 android.app.Activity.onStart() method 中检查一些系统本身的 feature 是否处于激活状态，因为用户可能会离开很长的时间。

#重新创建一个 android.app.Activity

一个 android.app.Activity 被销毁，可能是由于以下情况：

- 用户按了 Back 按钮  
- android.app.Activity 自身调用了 android.app.Activity.finish() method  
- 系统因为需要释放资源而销毁了该 android.app.Activity  

在前两种情况下，系统会认为这个 android.app.Activity 已经永远消失；在第三种情况下，系统会记住这个 android.app.Activity 曾经存在过，当用户按返回键的时候会重新创建它。
