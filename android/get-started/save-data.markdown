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

内部存储                         | 外部存储                                                                       
---------------------------------|--------------------------------------------------------------------------------
永远可用                         | 有时候不可用                                                                   
这里存储的文件只有开发者可以访问 | 别的程序也可以访问                                                             
当用户卸载程序时，系统会自动删除 | 只有存储在 android.content.Context.getExternalFilesDir() 目录下的文件才会被删除

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

###在外部设备上存储文件

在每次访问外部存储设备的时候，开发者都应该通过 android.os.Environment.getExternalStorageState() 检测外部存储设备的状态，只有当它的返回值是 android.os.Environment.MEDIA\_MOUNTED，才可以读写文件。

android 外部存储设备上可以存储两种文件： 

- public 文件。用户卸载程序的时候，不会被删除
- private 文件。用户卸载程序的时候，会被删除

如果想要在外部存储设备上保存 public 文件，开发人员需要调用 android.os.Environment.getExternalStoragePublicDirectory() method 来获取一个 java.io.File object，这个 method 有一个参数，可以设置为 android.os.Environment.DIRECTORY\_MUSIC 或 android.os.Environment.DIRECTORY\_PICTURES，可以额外指定要存出的文件的类型，比如音乐或图片，以使系统可以更有逻辑地存放这些文件。

如果想要在外部存储设备上保存 private 文件，则需要调用 android.os.Context.getExternalFilesDir() method。这个 method 同样可以使用参数来指定文件类型，如果所有指定的文件类型都没有对应的现有的目录存储，则需要将参数替换成 null，这样就可以获取 private 的根目录。

###查询可用的存储空间

如果事先知道需要存储的文件的大小，开发者可以先使用 java.io.File.getFreeSpace() 或 java.io.getTotalSpace() methods 查询系统中是否有足够的可用空间，以免产生 java.io.IOException。

需要注意的是，使用系统并不能保证使用 java.io.File.getFreeSpace() method 获取到的大小的空间都是可用的。

###删除文件

删除文件的方式有两种，一种是直接在 java.io.File 上调用 delete() method，另外一种方式是使用 android.content.Context.deleteFile(fileName) method;

当用户卸载程序到时候，系统会自动删除内部存储设备上的文件以及使用 android.content.Context.getExternalFilesDir() method 在外部存储设备上保存的文件。开发者需要定期清理使用 android.content.Context.getCacheDir() method 保存的缓存文件。

##在 SQL 数据库中保存数据

###定义 Schema 和 Contract

Schema 是数据库设计结构的 declaration。contract class 可以用一种较为系统的方式表示 Schema。

contract class 包含了一些常量，包含数据库表的名、数据库表栏的名以及一些 URI。

一个创建 contract class 的好方法是把整个数据库都能用到的常量放置在 class 的顶级，然后在 class 内为每个表创建一个包含这个表的栏名的 inner class。

当实现 android.provider.BaseColumns 接口时，内部 class 自动继承 \_ID 成员。

	public static abstract class FeedEntry implements BaseColumns {
		public static final String TABLE_NAME = "entry";
		public static final String COLUMN_ENTRY_ID = "entryid";
		public static final String COLUMN_NAME_TITLE = "title";
		...
	}

###使用 SQL Helper 创建数据库

首先，定义一些有关 SQL 操作的语句：

	private static final String TEXT_TYPE = " TEXT ";
	private static final String COMMA_SEP = ",";
	private statid final String SQL_CREATE_ENTRIES = 
			" CREATE TABLE " + FeedReaderContract.FeedEntry.TABLE_NAME + " (" +
			FeedReaderContract.FeedEntry._ID + " INTEGER PRIMARY KEY," +
			FeedReaderContract.FeedEntry.COLUMN_NAME_ENTRY_ID + TEXT_TYPE + COMMA_SEP +
			FeedReaderContract.FeedEntry.COLUMN_NAME_TITLE + TEXT_TYPE + COMMA_SEP +
			...
			" )";
	private static final String SQL_DELETE_ENTRIES = "DROP TABLE IF EXISTS " + TABLE_NAME_ENTRIES;

android.database.sqlite.SQLiteOpenHelper class 提供了一系列有用的 API。当开发者使用这个 class 的时候，系统会根据资源占用自动调度数据库操作，开发者只需要在后台线程调用 android.database.sqlite.SQLiteOpenHelper.getWritableDatabase() 以及 android.database.SQLiteOpenHelper.getReadableDatabase() methods 完成工作。

	public class FeedReaderDbHelper extends SQLiteOpenHelper {
		public static final int DATABASE_VERSION = 1;
		public static final String DATABASE_NAME = "FeedReader.db";

		public FeedReaderDbHelper(Context context) {
			super(context, DATABASE_NAME, null, DATABASE_VERSION);
		}

		public void onCreate(SQLiteDatabase db) {
			db.execSQL(SQL_CREATE_ENTRIES);
		}

		public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
			db.execSQL(SQL_DELETE_ENTRIES);
			onCreate(db);
		}

		public void onDowngrade(SQLiteDatabase db, int oldVersion, int newVersion) {
			onUpgrade(db, oldVersion, newVersion);
		}
	}

如果需要访问数据库，可以通过这样的方式访问：

	FeedReaderDbHelper mDbHelper = new FeedReaderDbHelper(getContext());

###向数据库中添加数据

开发者可以将一个 android.context.ContentValues object 传递给 android.sqlite.SQLiteDatabase.insert() method 来向数据库中添加数据。

	SQLiteDatabase db = mDbHelper.getWritableDatabase();
	ContentValues values = new ContentValues();
	values.put(FeedReaderContract.FeedEntry.COLUMN_NAME_ENTRY_ID, id);
	values.put(FeedReaderContract.FeedEntry.COLUMN_NAME_TITLE, title);
	values.put(FeedReaderContract.FeedEntry.COLUMN_ENTRY_CONTENT, content);
	long newRowId;
	newRowId = db.insert(
			FeedReaderContract.FeedEntry.TABLE_NAME,
			FeedReaderContract.FeedEntry.COLUMN_NAME_NULLABLE,
			values);

###丛数据库中读取数据

想从数据库中读取数据，需要使用 android.database.sqlite.SQLiteDatabase.query() method，这个 method 还融合了 android.database.sqlite.SQLiteDatabase.insert() 以及 android.database.sqlite.SQLiteDatabase.update() 两个 methods 的元素，不同的是，android.database.sqlite.SQLiteDatabase.query() method 提供的是获取数据的方法而不是更改数据。这个 method 的返回值是 android.database.Cursor object。

	SQLiteDatabase db = mDbHelper.getReadableDatabase();

	String[] projection = {
			FeedReaderContract.FeedEntry._ID,
			FeedReaderContract.FeedEntry.COLUMN_NAME_TITLE,
			FeedReaderContract.FeedEntry.COLUMN_NAME_UPDATED,
			...
	};

	String sortOrder = FeedReaderContract.FeedEntry.COLUMN_NAME_UPDATED + " DESC ";
	
	Cursor c = db.query(
		FeedReaderContract.FeedEntry.TABLE_NAME,
		projection,
		selection,
		selectionArgs,
		null,
		null,
		sortOrder
	);

如果需要读取某一栏的内容，必须先使用 android.database.Cursor object 的 move methods。一般来说，开发者需要先调用 android.database.Cursor.moveToFirst() method 开始。开发人员可以使用 android.database.Cursor.getString() 和 android.database.Cursor.getLong() 等 methods，使用这些方法的时候，必须传递栏的 index，后者可以通过调用 android.database.Cursor.getColumnIndex() 或 getColumnIndexOrThrow() methods 获取：

	cursor.moveToFirst();
	long itemId = cursor.getLong(
		cursor.getColumnIndexOrThrow(FeedReaderContract.FeedEntry._ID);
	);

###从数据库中删除信息

	String selection = FeedReaderContract.FeedEntry.COLUMN_NAME_ENTRY_ID + " LIKE ?";
	String[] selectionArgs = { String.valueOf(rowId) };
	db.delete(table_name, selection, selectionArgs);

###更新数据库

当需要更新数据库中的一部分数值时，可以调用 android.database.sqlite.SQLiteDatabase method：

	SQLiteDatabase db = mDbHelper.getReadableDatabase();

	ContentValues values = new ContentValues();
	values.put(FeedReaderContract.FeedEntry.COLUMN_NAME_TITLE, title);

	String selection = FeedReaderContract.FeedEntry.COLUMN_NAME_ENTRY_ID + " LINKE ? ";
	String[] selectionArgs = { String.valueOf(rowId) };

	int count = db.update(
			FeedReaderDbHelper.FeedEntry.TABLE_NAME,
			values,
			selection,
			selectionArgs);
