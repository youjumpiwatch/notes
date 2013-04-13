#启动一个 activity

##响应发送按钮

为了响应按钮的点击事件，为 activity\_main.xml 文件增加 android:onClick 属性

	<Button
		...
		android::onClick="sendMessage" />

onClick 的属性值 sendMessage 是当前 activity 的 method 名称，它必须满足以下条件：

- access label 是 public  
- 返回值是 void  
- 只有一个参数，类型是 [android.view.View](http://developer.android.com/reference/android/view/View.html)  

##创建并启动一个 android.content.Intent

android.content.Intent 的 public constructor 有：

	Intent();
	Intent(Intent o);
	Intent(String action);
	Intent(String action, Uri uri);
	Intent(Context packageContext, Class<?> cls);
	Intent(String action, Uri uri, Context packageContext, Class<?> cls);

一个 [android.content.Intent](http://developer.android.com/reference/android/content/Intent.html) 提供了组件间的 runtime binding。一般用来启动另外一个 activity。可以通过 android.content.Intent 的 constructor 来创建一个 android.content.Intent

	Intent intent = new Intent(this, DisplayMessageActivity.class);

由于 android.app.Activity 类是 *Context* 类的子类(java.lang.Object -> android.content.Context -> android.content.ContextWrapper -> android.view.contectThemeWrapper -> android.app.Activity)，所以可以用于参数是 *Context* 类的 method 中

一个 android.content.Intent 不但可以用来启动另外一个 android.app.Activity，也可以用来承载一系列的数据，如：

	public final static String EXTRA_MESSAGE = "com.example.app.MESSAGE"
	Intent intent = new Intent(this, DisplayMessageActivity.class);
	EditTExt editText = (EditTExt)findViewByID(R.id.edit_message);
	String message = editText.getText().toString();
	intent.putExtra(EXTRA_MESSAGE, message);
	startActivity(intent);

一个 android.content.Intent 可以承载一系列的 Key-Value 数据，叫做 *extras*。android.content.Intent.putExtra() method 的第一个参数是 Key，第二个参数是 Value。习惯上，Key 一般用 app 的名称做前缀

##创建一个 android.app.Activity

android.app.Activity 的所有子类必须实现 android.app.Activity.onCreate() method，系统在创建 android.app.Activity 的时候会自动调用这个 method。

	public class DisplayMessageActivity extends Activity {

		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_display_message);

			if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
				getActionBar().setDisplayHomeAsUpEnabled(true);
			}
		}

		public boolean onOptionsItemSelected(MenuItem item) {
			switch (item.getItemId()) {
				case android.R.id.home:
					NavUtils.navigateUpFromSameTask(this);
					return true;
			}
			return super.onOptionsItemSelected(item);
		}
	}

在这个 android.app.Activity 中，使用到了 android.view.View 类的 object R.layout.activity_display_message ，需要在资源文件 strings.xml 中为该 *View* 添加标题：

	<resources>
		...
		<string name="title_activity_display_message">My Message</string>
	</resources>

所有的 android.app.Activity 必须在 manifest 文件里声明，所以需要把刚刚创建的 android.app.Activity 添加到 AndroidManifest.xml 中去。

	<application ...>
		...
		<activity
			android:name="com.example.app.DisplayMessageActivity"
			android:label="@string/title_activity_display_message"
			android:parentActivityName="com.example.app.MainActivity" >
			<meta-data
				android:name="android.support.PARENT_ACTIVITY"
				android:value="com.example.app.MainActivity" />
		</activity>
	</application>

##接收一个 android.content.Intent

每个 android.app.Activity 都是被一个 android.content.Intent启动的。在被启动的 android.app.Activity 中可以调用 [getIntent()](http://developer.android.com/reference/android/app/Activity.html#getIntent(\))

	Intent intent = getIntent();
	String message = intent.getStringExtra(MainActivity.EXTRA_MESSAGE);
