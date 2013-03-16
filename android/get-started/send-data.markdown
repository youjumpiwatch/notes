#分享数据

##向其他程序发送数据

当开发者创建 android.content.Intent 的时候，必须指定这个 android.content.Intent 要执行的 action。要想发送出数据，开发者只需要指定数据及其类型，系统会自动识别可作出响应的 android.app.Activity，并将数据发送给它。同样地，也可以在 manifest 文件中指定自己的 android.app.Activity 可以响应的数据类型。

为 android.app.ActionBar 添加 share action 的最佳方法是使用 android.widget.ShareActionProvider。

###发送文字

android.content.Intent.ACTION\_SEND action 的最常见用法是从一个 android.app.Activity 向另外一个 android.app.Activity 发送数据。

	Intent sendIntent = new Intent();
	sendIntent.setAction(Intent.ACTION_SEND);
	sendIntent.putExtra(Intent.EXTRA_TEXT, "This is my text");
	sendIntent.setType("text/plain");
	startActivity(Intent.createChooser(sendIntent, getResources().getText(R.string.send_to)));

###发送 binary 数据

可以通过指定 MIME 类型，并将 binary 数据放置在 Intent 的 extra 数据区来发送 binary 数据

	Intent shareIntent = new Intent();
	shareIntent.setAction(Intent.ACTION_SEND);
	shareIntent.putExtra(Intent.EXTRA_STREAM, uriToImage);
	shareIntent.setType("image/jpeg");
	startActivity(Intent.createChooser(shareIntent, getResources().getText(R.string.send_to)));

###发送多条数据

结合多个 URI，android.content.Intent.ACTION\_SEND\_MULTIPLE action 可以用来一次性发送多条数据。如果数据类型都是图片，可以将 MIME 设置为 "image/\*"，如果是混杂类型，可以设置为 “\*/\*”

	ArrayList<Uri> imageUris = new ArrayList<Uri>();
	imageUris.add(imageUri1);
	imageUris.add(imageUri2);
	...

	Intent shareIntent = new Intent();
	shareIntent.setAction(Intent.ACTION_SEND_MULTIPLE);
	shareIntent.putParcelabelArrayListExtra(Intent.EXTRA_STREAM, imageUris);
	shareIntent.setType("image/*");
	startActivity(Intent.createChooser(shareIntent, "Share images to ..."));

##从其他程序接收数据

android.content.Intent filters 会通知系统当前的 android.app.Activity 可以接收那些类型的数据。

	<activity android:name=".ui.MyActivity" >
		<intent-filter>
			<action android:name="android.intent.action.SEND" />
			<category android:name="android.intent.category.DEFAULT" />
			<data android:mimeType="image/*" />
		</intent-filter>
		<intent-filter>
			<action android:name="android.intent.action.SEND" />
			<category  android:name="android.intent.category.DEFAULT" />
			<data android:mimeType="text/plain" />
		</intent-filter>
		<intent-filter>
			<action android:name="android.intent.action.SEND_MULTIPLE" />
			<category  android:name="android.intent.category.DEFAULT" />
			<data android:mimeType="image/*" />
		</intent-filter>
	</activity>

##增加一个 easy share action

###增加菜单声明

在使用 android.widget.ShareActionProvider 之前，先要在菜单文件里添加一些内容：

	<menu xmlns:android="http://schemas.android.com/apk/res/android">
		<item android:id="@+id/menu_item_share"
			android:showAsAction="ifRoom"
			android:title="Share"
			android:actionProviderClass="android.widget.ShareActionProvider" />
		...
	</menu>

###设置 share android.content.Intent

开发者必须为 android.widget.ShareActionProvider 提供一个 android.content.Intent。

要想设置一个 share android.widget.Intent，需要：

- 设置 Intent 的 action 为 ACTION\_SEND，如有需要，设置 data 类型
- 首先获取 android.view.MenuItem
- 使用 android.view.MenuItem.getActionProvider() 获取 android.widget.ShareActionProvider 的 instance。
- 使用 android.widget.ShareActionProvider.setShareIntent() 更新 share android.content.Intent

	private ShareActionProvider mShareActionProvider;

	public boolean onCreateOptionsMenu(Menu menu) {
		getMenuInflater().inflate(R.menu.share_menu, menu);
		MenuItem item = menu.findItem(R.id.menu_item_share);
		mShareActionProvider = (ShareActionProvider) item.getActionProvider();
		return true;
	}

	private void setShareIntent(Intent shareIntent) {
		if (mShareActionProvider != null) {
			mShareActionProvider.setShareIntent(shareIntent);
		}
	}
