#创建动态的 UI

为了创建动态的、多面板的 UI，开发者需要把 UI 组件以及 android.app.Activity 封装到模块中。开发者可以使用 android.app.Fragment class 去创建这些模块。

##使用 Support 库

android 的 Support 库是一个 jar 文件，为老设备提供一些新的 API。

###为工程加上 Support 库

对应于 API level 4 的库的地址在：

	<sdk>/extras/android/support/v4/android-support-v4.jar

这样就可以更新 manifest 文件，把工程支持的最低 API level 调整为 4：

	<uses-sdk android:minSdkVersion="4" android:targetSdkVersion="15" />

###引入 Support 库中的 API

	import android.support.v4.app.Fragment;
	import android.support.v4.app.FragmentManager;

##创建一个 android.app.Fragment

开发者可以把 android.app.Fragment 视为 android.app.Activity 的一部分；它有自己的 lifecycle、响应自身的输入事件，在 android.app.Activity 运行的时候，可以添加或者删除多个 android.app.Fragment。

###创建一个 android.app.Fragment class

如果想要创建一个 android.app.Fragment ，可以通过 extend android.app.Fragment class，然后 override 主要的 lifecycle methods。这与创建 android.app.Activity 类似。

但二者也是存在区别的，比如创建 android.app.Fragment 的时候，必须使用 android.app.Fragment.onCrateView() 回调函数去定义 layout:

	import android.os.Bundle;
	import android.support.v4.app.Fragment;
	import android.view.LayoutInflater;
	import android.view.ViewGroup;

	public class ArticleFragment extends Fragment {
		@Override
		public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
			return inflater.inflate(R.layout.article_view, container, false);
		}
	}

像 android.app.Activity 一样，一个 android.app.Fragment 也需要实现其他的 lifecycle 回调函数，这些回调函数应用于该 android.app.Fragment 增加到 android.app.Activity 或者从 android.app.Activity 删除时，以及它所属的 android.app.Activity 自身的状态转变。比如，当系统调用了它所属的 android.app.Activity 的 android.app.Activity.onPause() 方法时，该 android.app.Activity 的所有 android.app.Fragment 都会接收到 android.app.Fragment.onPause() 调用。

android.app.Fragment 是模块化的、可以重复利用的 UI 组件，每个 android.app.Fragment 的 instance 必须关联到一个 parent android.app.Activity/android.app.support.v4.FragmentActivity。这可以通过在 layout 的 XML 文件中加以定义实现。如：

文件：

	res/layout-large/news_articles.xml

内容：

	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android" ...>
		<fragment android:name="com.example.android.fragments.HeadlinesFragment" ... />
		<fragment android:name="com.example.android.fragments.ArticleFragment" ... />
	</LinearLayout>

对于 API level 4 至 API level 11 的设备，可以通过这样的方式来创建包含 android.app.Fragment 的 android.app.Activity：

	import android.os.Bundle;
	import android.support.v4.app.FragmentActivity;

	public class MainActivity extends FragmentActivity {
		public void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.news_articles);
		}
	}

值得注意的是，如果是通过在 layout XML 向指定的 android.app.Activity 添加了 android.app.Fragment，那么在程序运行的时候，开发者是不能从指定的 android.app.Activity 中移除 android.app.Fragment 的。如果想要更换 android.app.Fragment ，则需要在启动的时候就把所需的 android.app.Fragment 添加到 android.app.Activity 中去。

##创建灵活的 UI

当为多种尺寸的屏幕开发程序时，可以在不同的 layout 里重复利用 android.app.Fragment 以优化用户体验：

![Alt flexible-UI](images/fragments-screen-mock.png "android flexible UI")

android.app.FragmentManager 提供了在运行时动态向 android.app.Activity 动态增加、删除、替换 android.app.Fragment 的功能

###在运行时，向 android.app.Activity 动态增加 android.app.Fragment

如果需要批量的添加、删除 android.app.Fragment ，则需要开发者使用 android.app.FragmentTransaction class，它提供了添加、删除、替换 Fragment 的功能。

如果开发者打算改变 android.app.Fragment ，则应该在 android.app.Activity 调用 android.app.Activity.onCreate() method 的时候，添加初始的 android.app.Fragment。

一条重要的原则是，如果开发者想要动态创建 android.app.Fragment，那么必须为它保留一个 android.view.View。

下面这个示例，演示了如何在 layout XML 中配置一个 android.app.Fragment 的 container android.widget.FrameLayout，以及如何向该 android.widget.FrameLayout 中增加 android.app.Fragment。

文件：

	res/layout/news_articles.xml

内容：

	<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
		android:id="@+id/fragment_container"
			android:layout_width="match_parent"
			android:layout_height="match_parent" />

代码：

	import android.os.Bundle;
	import android.app.Activity;

	public class MainActivity extends Activity {
		public void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.news_articles);

			if (findViewById(R.id.fragment_container) != null) {
				if (savedInstanceState != null) {
					return;
				}
				HeadlinesFragment firstFragment = new HeadlinesFragment();
				firstFragment.setArguments(getIntent().getExtras());
				getFragmentManager().beginTransaction().add(R.id.fragment_container, firstFragment).commit();
			}
		}
	}

###替换掉一个 android.app.Fragment

如果需要替换掉一个 android.app.Fragment，只需要把 android.app.FragmentTransaction.add() method 替换成 android.app.FragmentTransaction.replace() method。

当开发者替换 android.app.Fragment 的时候，应该注意，或许用户想要通过返回按钮来撤销操作，这个时候，就应该在调用 android.app.FragmentTransaction.commit() method 之前调用 android.app.FragmentTransaction.addToBackStack() method 

	ArticleFragment newFragment = new ArticleFragment();
	Bundle args = new Bundle();
	args.putInt(ArticleFragment.ARG_POSITION, position);
	newFragment.setArguments(args);
	FragmentTransaction transaction = getFragmentManager().beginTransaction();
	transaction.replace(R.id.fragment_container, newFragment);
	transaction.addToBackStack(null);
	transaction.commit();

##android.app.Fragment 间的通信

所有的 android.app.Fragment 到 android.app.Fragment 间的通信都应该通过它所属的 android.app.Activity 来完成。android.app.Fragment 不能直接通信。

###定义接口

为了使两个 android.app.Fragment 可以通信，开发者可以在它们所属的 android.app.Activity 中定义一个接口。android.app.Fragment 会在调用 android.app.Fragment.onAttach() 时发现这个接口，并使用它进行通信：

	public class HeadlinesFragment extends ListFragment {
		OnHeadlineSelectedListener mCallback;

		public interface OnHeadlineSelectedListener {
			public void onArticleSelected(int position);
		}

		public void onAttach(Activity activity) {
			super.onAttach(activity);

			try {
				mCallback = (OnHeadlineSelectedListener) activity;
			} catch (ClassCastException e) {
				throw new ClassCastException(activity.toString() + " must implement OnHeadlineSelectedListener");
			}
		}
	}

这时，android.app.Fragment 就可以通过调用 onArticleSelected() method 向 android.app.Activity 发送消息。

	public void onListItemClick(ListView l, View v, int position, long id) {
		mCallback.onArticleSelected(position);
	}

###落实接口

为了从 android.app.Fragment 接收事件的 callback 函数，android.app.Activity 必须落实在 android.app.Fragment 中定义的接口。

	public static class MainActivity extends Activity
			implements HeadlinesFragment.OnHeadlineSelectedListener {
		...

		public void onArticleSelected(int position) {
			...
		}
	}

###向 android.app.Fragment 发送消息

android.app.Activity 可以通过 android.app.FragmentManager.findFragmentById() 查找到 android.app.Fragment 并发送消息；

	public static class MainActivity extends Activity
			implements HeadlinesFragment.OnHeadlineSelectedListener{
		...

		public void onArticleSelected(int position) {
			ArticleFragment articleFrag = (ArticleFragment) getFragmentManager().findFragmentById(R.id.article_fragment);
			if (articleFrag != null) {
				articleFrag.updateArticleView(position);
			} else {
				ArticleFragment newFragment = new ArticleFragment();
				Bundle args = new Bundle();
				args.putInt(ArticleFragment.ARG_POSITION, position);
				newFragment.setArguments(args);
				FragmentTransaction transaction = getFragmentManager().beginTransaction();
				transaction.replace(R.id.fragment_container, newFragment);
				transaction.addToBackStack(null);
				transaction.commit();
			}
		}
	}
