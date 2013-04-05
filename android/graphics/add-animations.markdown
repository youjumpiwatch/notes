#增加动画

##层叠 view

###创建 android.view.View

	<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
		android:layout_width="match_parent"
		android:layout_height="match_parent">
	
		<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
			android:id="@+id/content"
			android:layout_width="match_parent"
			android:layout_height="match_parent">
	
			<TextView style="?android:textAppearanceMedium"
				android:lineSpacingMultiplier="1.2"
				android:layout_width="match_parent"
				android:layout_height="wrap_content"
				android:text="@string/lorem_ipsum"
				android:padding="16dp" />
		</ScrollView>
	
		<ProgressBar android:id="@+id/loading_spinner"
			style="?android:progressBarStyleLarge"
			android:layout_width="wrap_content"
			android:layout_height="wrap_content"
			android:layout_gravity="center" />
	</FrameLayout>

###设置动画

主要分两步：

- 将主要的 android.view.View 设置为 android.view.View.GONE
- 缓存 android.R.integer.config\_shortAnimTime 或者 android.R.integer.config\_longAnimTime

代码：

	public class CrossfadeActivity extends Activity {
	
		private View mContentView;
		private View mLoadingView;
		private int mShortAnimationDuration;
	
		...
	
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_crossfade);
	
			mContentView = findViewById(R.id.content);
			mLoadingView = findViewById(R.id.loading_spinner);
	
			mContentView.setVisibility(View.GONE);
	
			mShortAnimationDuration = getResources().getInteger(android.R.integer.config_shortAnimTime);
		}

		...
	}

###淡入淡出效果

设置淡入淡出效果需要三步：

- 要淡入的 android.view.View，需要将 alpha 值设置为 0，并将可视属性设置为 android.view.View.VISIBLE
- 要淡入的 android.view.View，需要将 alpha 值从 0 变到 1；同时，需要淡出的 android.view.View，需要将其 alpha 值从 1 变到 0
- 使用 android.animation.Animator.AnimatorListener.onAnimationEnd() method，将要淡出的 android.view.View 的可视属性设置为 android.view.View.GONE

代码：

	private View mContentView;
	private View mLoadingView;
	private int mShortAnimationDuration;
	
	private void crossfade() {
		mContentView.setAlpha(0f);
		mContentView.setVisibility(View.VISIBLE);
	
		mContentView.animate().alpha(1f).setDuration(mShortAnimationDuration).setListener(null);
	
		mHideView.animate()
				.alpha(0f)
				.setDuration(mShortAnimationDuration)
				.setListener(new AnimatorListenerAdapter() {
					@Override
					public void onAnimationEnd(Animator animation) {
						mHideView.setVisibility(View.GONE);
					}
				});
	}

##使用 android.support.v4.view.ViewPager

android.support.v4.view.ViewPager class 可以实现页面的滑动切换
