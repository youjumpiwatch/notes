#保存数据

很多 android 程序都需要 android.app.Activity 调用 android.app.Activity.onPuase() method 时保存数据，或者保存用户的一些偏好设置。android 系统提供多种形式的数据存储：

- 在 shared preferences 文件里保存 Key-Value 数据
- 在文件里系统里保存文件
- 使用 SQLite 数据库

##保存 Key-Value 集合

如果程序只需要保存少量的数据，则应该使用 android.content.SharedPreferences API。一个 android.content.SharedPreferences object 对应于一个包含很多 Key-Value 集合的文件，并提供读写功能。每个 android.content.SharedPreferences 文件都由 android 框架管理，属性可以是 private 也可以是 shared。

android.content.SharedPreferences 与 android.preference.Preference 不同，后者是为了帮助开发者建立程序设置界面的 class。

###获取一个 android.content.SharedPreferences handle

开发者可以新建一个 android.content.SharedPreferences，也可以访问现有的 android.content.SharedPreferences，两种方法如下：

- android.content.Context.getSharedPreferences()。如果需要使用多个 android.content.SharedPreferences 文件，那么就应该用这个 method
- android.app.Activity.getPreferences()，如果只需要使用一个 android.content.SharedPreferences 文件，那么就应该用这个 method。

	Context context = getActivity();
	SharedPreferences sharedPref = context.getSharedPreferences(getString(R.string.preference_file_key), Context.MODE_PRIVATE);

如果开发者使用 android.content.Context.MODE\_WORLD\_READABLE 或者 android.content.Context.MODE\_WORLD\_PRIVATE 参数，那么任何一个知道 android.content.SharedPreferences 文件名的程序都可以访问它里面的数据。

###写入 android.content.SharedPreferences 文件

如果需要写入 android.content.SharedPreferences 文件，则需要使用 android.content.SharedPreferences.Editor.edit() 方法。

	SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
	SharedPreferences.Editor editor = sharedPref.edit();
	editor.putInt(getString(R.string.saved_high_score), newHighScore);
	editor.commit();

###从 android.content.SharedPreferences 文件中读取数据

	SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
	int defaultValue = getResources().getInteger(R.string.saved_high_score_default);
	long highScore = sharedPref.getInt(getString(R.string.saved_high_score), defaultValue);

##保存文件

android 使用 java.io.File class 来进行文件操作。

###选择内部或者外部存储

所有的 android 设备都提供内部存储和外部存储。

----------------------------------------------------------------------------------------------------------------------
| 内部存储                         | 外部存储                                                                        |
----------------------------------------------------------------------------------------------------------------------
| 永远可用                         | 有时候不可用                                                                    |
----------------------------------------------------------------------------------------------------------------------
| 这里存储的文件只有开发者可以访问 | 别的程序也可以访问                                                              |
----------------------------------------------------------------------------------------------------------------------
| 当用户卸载程序时，系统会自动删除 | 只有存储在 android.content.Context.getExternalFilesDir() 目录下的文件才会被删除 |
----------------------------------------------------------------------------------------------------------------------

开发者可以在 manifest 文件里，使用 android:installLocation 属性指定程序将要被安装在内部存储设备还是外部存储设备中

###设置外部设备存储权限

开发者需要在 manifest 文件里手动设置外部存储的的权限才能读写外部存储设备：

	<manifest ... >
		<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
		<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
	</manifest>

开发者无须设置特殊权限就可以直接使用设备内部存储设备。

###将文件存储到内部设备

开发者可以使用两个 methods 来获取内部存储目录：

- android.content.Context.getFilesDir() 返回一个 java.io.File object，表示一个内部存储目录
- android.content.Context.getCacheDir() 返回一个 java.io.File object，表示一个临时缓存目录，开发者需要记着删掉这个目录

下面的代码可以用来创建一个新文件

	File file = new File(context.getFilesDir(), filename);

或者，也可以调用 android.content.Context.openFileOutput() method 来获取一个 java.io.FileOutputStream object。

	String filename = "myfile";
	String string = "Hello, world!";
	FileOutputStream outputStream;

	try {
		outputStream = openFileOutput(filename, Context.MODE_PRIVATE);
		outputStream.write(string.getBytes());
		outputStream.close();
	} cache (Except e) {
		e.printStackTrace();
	}

如果仅仅是需要缓存一些文件，则需要使用 java.io.File.createTempFile() method：

	public File getTempFile(Context context, String url) {
		File file;
		try {
			String fileName = Uri.parse(url).getLastPathSegment();
			file = File.createTempFile(fileName, null, context.getCacheDir());
		} cache (IOException e) {
		}
		return file;
	}
