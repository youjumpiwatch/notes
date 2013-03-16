#程序间互动

##启动其他程序

android 系统的一个很重要的特性是，它可以按照想要执行的 action 启动另外一个程序。当开发者需要在程序内部切换的时候，需要使用 explicit android.content.Intent，当开发者需要启动一个独立的程序时，则需要使用 implicit android.content.Intent。

###创建一个 implicit android.content.Intent

explicit android.content.Intent 使用 class 的名字来启动 android.app.Activity，implicit android.content.Intent 则使用 action 来启动程序，这个 action 可以是 view，edit，send 或者 get 等等，开发者一般还需要把数据传送给 action。

	Uri number = Uri.parse("tel:5555");
	Intent callIntent = new Intent(Intent.ACTION_DIAL, number);

当程序执行到这段代码的时候，系统会自动启动拨号界面。类似的行为还有，发送一个邮件：

	Intent emailIntent = new Intent(Intent.ACTION_SEND);
	emailIntent.setType(HTTP.PLAIN_TEXT_TYPE);
	emailIntent.putExtra(Intent.EXTRA_EMAIL, new String[] { "jon@example.com"} );
	emailIntent.putExtra(Intent.EXTRA_SUBJECT, "Email subject");
	emailIntent.putExtra(Intent.EXTRA_TEXT, "Email message text");
	emailIntent.putExtra(Intent.EXTRA_STREAM, Uri.parse("content://path/to/email/attachment"));

开发者可以使用 android.app.Activity.startActivity() 来调用 Intent

###检查系统是否有程序响应 android.content.Intent

开发者需要调用 android.content.pm.PackageManager.queryIntentActivities() 来获取一个可响应指定 android.content.Intent 的程序列表。

	PackageManager packageManager = getPackageManager();
	List<ResolveInfo> activities = packageManager.queryIntentActivities(intent, 0);
	boolean isIntentSafe = activities.size() > 0;

###启动 android.content.Intent

	Uri location = Uri.parse("geo:0,0?q=1600+Amphitheatre+Parkway,+Mountain+View,+California");
	Intent mapIntent = new Intent(Intent.ACTION_VIEW, location);

	PackageManager packageManager = getPackageManager();
	List<ResolveInfo> activities = packageManager.queryIntentActivities(mapIntent, 0);
	boolean isIntentSafe = activities.size() > 0;

	if (isIntentSafe) {
		startActivity(mapIntent);
	}

###调用打开程序列表

当有多个程序可以响应 android.content.Intent 时，用户可以通过列表选择。

开发者可以通过调用 android.content.Intent.createChooser() 创建一个打开程序列表：

	Intent intent =	new Intent(Intent.ACTION_SEND);

	String title = getResources().getText(R.string.chooser_title);
	Intent chooser = Intent.createChooser(intent, title);
	startActivity(chooser);

##从其他 android.app.Activity 获取运行结果

如果想要获取其他 android.app.Activity 的运行结果，则应该使用 android.app.startActivityForResult() method 而不是 android.app.Activity.startActivity() method。

目标 Activity 应该在设计时就考虑到了设定返回值，它会以 android.content.Intent object 的形式返回，开发者可以在当前 android.app.Activity 上使用 android.app.Activity.onActivityResult() 回调函数获取。

尽管 explicit 和 implicit 类型的 android.content.Intent 都可以使用 android.app.Activity.startActivityForResult() method 启动，当开发者在调用自己的 android.app.Activity 的时候，应该使用 explicit android.content.Intent，以免获取的结果有差错。

###启动一个 android.app.Activity

当调用 android.app.Activity.startActivityForResult() 的时候，开发者需要传递一个额外的整型参数作为请求 ID，回调函数需要提供相同的请求 ID，以便程序可以将返回值传递给正确的回调函数。

	static final int PICK_CONTACT_REQUEST = 1;
	...
	private void pickContact() {
		Intent pickContactIntent = new Intent(Intent.ACTION_PICK, new Uri("content://contacts"));
		pickContactIntent.setType(Phone.CONTENT_TYPE);
		startActivityForResult(pickContactIntent, PICK_CONTACT_REQUEST);
	}

###接收 android.app.Activity 的返回值

当用户使用完被调用的 android.app.Activity，并返回的时候，系统会自动调用预先设定的 android.app.Activity.onActivityResult() method，这个 method 包含三个参数：

- 先前调用 android.app.Activity.startActivityForResult() 时设置的请求 ID
- 一个结果代码，可能是 android.app.Activity.RESULT\_OK 或者 android.app.Activity.RESULT\_CANCELED，取决于用户使用了被调用的 android.app.Activity 还是按了取消按钮。
- 一个携带结果数据的 android.content.Intent

代码：

	protected void onActivityResult(int requestCode, int resultCode, Intent data) {
		if (requestCode == PICK_CONTACT_REQUEST) {
			if (resultCode == RESULT_OK) {
				Uri contactUri = data.getData();
				String[] projection = { Phone.NUMBER };
				Cursor cursor = getContentResolver().query(contactUri, projection, null, null, null);
				cursor.moveToFirst();
				int column = cursor.getColumnIndex(Phone.NUMBER);
				String number = cursor.getString(column);
			}
		}
	}

##允许自己的程序被别的程序启动

如果想要允许其他程序调用自己的程序，那么应该在 manifest 文件的 <activity> 部分增加 <intent-filter> 。

###增加一个 android.content.Intent filter

系统把 android.content.Intent object 可以响应的 filter 划分为三类：

- action，如 ACTION\_SEND，ACTION\_VIEW。可以通过为 <intent-filter> 增加 <action> 元素来指定。
- data，可以通过为 <intent-filter> 增加 <data> 元素来指定，其属性值可以取 MIME，URI，URI scheme，或者是他们的排列组合。
- category，通常与用户的操作手势以及被调用的 android.app.Activity 启动位置有关。系统提供了很多种 categories，所有的 implicit android.content.Intent 都被定义为 CATEGORY\_DEFAULT。可以通过为 <intent-filter> 增加 <category> 元素来指定。

内容：

	<activity android:name="ShareActivity">
		<intent-filter>
			<action android:name="android.intent.action.SEND" />
			<category android:name="android.intent.category.DEFAULT" />
			<data android:mimeType="text/plain" />
			<data android:mimeType="image/*" />
		</intent-filter>
	</activity>

当需要定义互斥的 filter 时，开发者需要定义两个不同的 android.content.Intent filter。

	<activity anroid:name="SharedActivity">
		<intent-filter>
			<action  android:name="android.intent.action.SENDTO" />
			<category  android:name="android.intent.category.DEFAULT" />
			<data android:scheme="sms" />
			<data android:scheme="smsto" />
		</intent-filter>

		<intent-filter>
			<action  android:name="android.intent.action.SEND" />
			<category  android:name="android.intent.category.DEFAULT" />
			<data android:mimeType="image/*" />
			<data android:mimeType="text/plain" />
		</intent-filter>
	</activity>

如果想要响应 implicit android.content.Intent，必须在 <intent-filter> 中定义 CATEGORY\_DEFAULT 属性。

###处理 android.content.Intent

为了确定该执行什么 action，开发者需要分析系统传递过来的 android.content.Intent，可以通过 android.app.Activity.getIntent() 来获取，一般在自己的 android.app.Activity 的 android.app.Activity.onCreate() 或者 android.app.Activity.onStart() 进行。

	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.main);

		Intent intent = getIntent();
		Uri data = intent.getData();

		if (intent.getType().indexOf("image/") != -1) {
			...
		} else if (intent.getType().equals("text/plain")) {
			...
		}
	}

###返回一个值

如果需要向调用自己的 android.app.Activity 返回一个值，只需要调用 android.app.Activity.setResult() 并指定结果 ID 以及用来存放返回数据的 android.content.Intent

	Intent result = new Intent("com.example.RESULT_ACTION", Uri.parse("content://result_uri");
	setResult(Activity.RESULT_OK, result);
	finish();
