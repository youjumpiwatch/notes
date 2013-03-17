#拍照

##快速拍照

###请求摄像头权限

在 manifest 文件里：

	<manifest ...>
		<uses-feature android:name="android.hardware.camera" />
		...
	</manifest>

如果摄像头并非必须的，可以通过增加 android:required="false" 属性来指定；程序运行期间，也可以调用 android.content.pm.PackageManager.hasSystemFeature(PackageManager.FEATRUE\_CAMERA) 来检查摄像头是否可用。

###使用摄像程序拍照

	public static boolean isIntentAvailable(Context context, String action) {
		final PackageManager packageManager = context.getPackageManager();
		final Intent intent = new Intent(action);
		List<ResolveInfo> list = packageManager.queryIntentActivities(intent, PackageManager.MATCH_DEFAULT_ONLY);
		return list.size() > 0;
	}

	private void dispatchTackPictureIntent(int actionCode) {
		Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
		startActivityForResult(takePictureIntent, actionCode);
	}

###查看照片

android 系统自带的摄像程序会把图片数据编码以后存放在返回的 android.content.Intent 中：

	private void handleSmallCameraPhoto(Intent intent) {
		Bundle extras = intent.getExtras();
		mImageBitmap = (Bitmap)extras.get("data");
		mImageView.setImageBitmap(mImageBitmap);
	}

###保存照片

	storageDir = new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES), getAlbumName());

###设置文件名

	private File createImageFile throws IOException {
		String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
		String imageFileName = JPEG_FILE_PREFIX + timeStamp + "_";
		File image = File.createTempFile(imageFileName, JPEG_FILE_SUFFIX, getAlbumDir());
		mCurrentPhotoPath = image.getAbsolutePath();
		return image;
	}

###在 android.content.Intent 里增加文件保存路径

	File f = createImageFile();
	takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, Uri.fromFile(f));

###把照片放到 gallery 中

	private void galleryAddPic() {
		Intent mediaScanIntent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
		File f = new File(mCurrentPhotoPath);
		Uri contentUri = Uri.fromFile(f);
		mediaScanIntent.setData(contentUri);
		this.sendBroadCast(mediaScanIntent);
	}

###更改照片尺寸

	private void setPic() {
		int targetW = mImageView.getWidth();
		int targetH = mImageView.getHeight();

		BitmapFactory.Options bmOptions = new BitmapFactory.Options();
		bmOptions.inJustDecodeBounds = true;
		BitmapFactory.decodeFile(mCurrentPhotoPath, bmOptions);
		int photoW = bmOptions.outWidth;
		int photoH = bmOptions.outHeight;

		int scaleFactor = Math.min(photoW/targetW, photoH/targetH);

		bmOptions.inJustDecodeBounds = false;
		bmOptions.inSampleSize = scaleFactor;
		bmOptions.inPurgeable = true;

		Bitmap bitmap = BitmapFactory.decodeFile(mCurrentPhotoPath, bmOptions);
		mImageView.setImageBitmap(bitmap);
	}
