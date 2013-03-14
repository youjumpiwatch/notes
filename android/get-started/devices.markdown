#多设备支持

##多语言支持

###创建本地化目录和字符串资源文件

如果需要支持多种语言，只需要在 res/ 目录下，复制一份 values 文件夹，并加上连词符以及 ISO 国家代码，比如：

	myProject/
		res/
			values/
				strings.xml
			values-es/
				strings.xml
			values-fr/
				strings.xml
			values-zh_CN/
				strings.xml

##多种屏幕支持

Android 设备的屏幕通常有两个属性：尺寸和密度。按照尺寸和密度划分，可以分为四类：

-尺寸: small, normal, large, xlarge
-密度: low(ldpi), medium(mdpi), high(hdpi), extra high(xhdpi)

同时，屏幕的放置方式也可以考虑成为屏幕尺寸的一种变种

-方向: landscape(land), portrait(port)

###创建尺寸的 android.R.layout

如果需要创建多种 android.R.layout ，需要在 res/ 目录下复制 layout 目录，并加上连词符和屏幕尺寸或方向，如：

	myProject/
		res/
			layout/
				main.xml
			layout-large/
				main.xml
			layout-land/
				main.xml
			layout-large-land/
				main.xml
###创建不同的 Bitmaps

为了使程序在不同屏幕上表现一致，开发者需要提供可以伸缩成四种密度(low, medium, high, extra-high density)的 bitmap 资源。为了生成这种图片，需要使用原始的矢量图按照如下倍数生成四种尺寸的图片：

-xhdpi: 2.0
-hdpi: 1.5
-mdpi: 1.0 (baseline)
-ldpi: 0.75

生成的图片应该放置在这样的目录里：

	myProject/
		res/
			drawable-xhdpi/
				image.png
			drawable-hdpi/
				image.png
			drawable-mdpi/
				image.png
			drawable-ldpi/
				image.png

##支持不同的平台

###指定最低 API 等级

AndroidManifest.xml 文件描述了程序执行的所需要的 SDK 信息：

	<manifest xmlns:android="http://schemas.android.com/apk/res/android" ... >
		<uses-sdk android:minSdkVersion="4" android:targetSdkVersion="15" />
		...
	</manifest>

###运行时检测平台版本信息

	private void setupActionBar() {
		if (android.os.Build.VERSION.SDK_INIT > android.os.Build.VERSION_CODES.HONEYCOMB) {
			android.app.ActionBar actionBar = getActionBar();
			actionBar.setDisplayHomeAsUpEnabled(true);
		}
	}

###使用平台的 Styles 和 Themes

如果想使 android.app.Activity 看起来像一个对话框：

	<activity android:theme="@android:style/Theme.Dialog">

如果想使 android.app.Activity 有一个透明的背景：

	<activity android:theme="@android.style/Theme.Translucent">

如果想使用 /res/values/styles.xml 中自定义的 theme：

	<activity android:theme="@style/CustomTheme">

如果想对所有的 android.app.Activity 都应用某个 theme：

	<application android:theme="@style/CustomTheme">
