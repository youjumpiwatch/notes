#拍照

##快速拍照

###请求摄像头权限

在 manifest 文件里：

	<manifest ...>
		<uses-feature android:name="android.hardware.camera" />
		...
	</manifest>

如果摄像头并非必须的，可以通过增加 android:required="false" 属性来指定；程序运行期间，也可以调用 android.content.pm.PackageManager.hasSystemFeature(PackageManager.FEATRUE\_CAMERA) 来检查摄像头是否可用。

###使用拍照程序拍照

	public static boolean isIntentAvailable(Context context, String action) {
		final PackageManager packageManager = context.getPackageManager();
		final Intent intent = new Intent(action);
		List<ResolveInfo> list = packageManager.queryIntentActivities(intent, PackageManager.MATCH_DEFAULT_ONLY);
		return list.size() > 0;
	}

	private void dispatchTakePictureIntent(int actionCode) {
		Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
		startActivityForResult(takePictureIntent, actionCode);
	}

###查看照片

android 系统自带的拍照程序会把图片数据编码以后存放在返回的 android.content.Intent 中：

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

##快速摄像

###请求摄像权限

在程序的 manifest 文件里添加：

	<manifest ... >
		<uses-feature android:name="android.hardware.camera" />
	</manifest>

如果摄像头不是必需的，可以添加 android:required="false" 属性；在运行时可以通过 android.content.pm.PackageManager.hasSystemFeature(PackageManager.FEATRUE\_CAMERA) 动态检测设备是否支持摄像。

###使用摄像程序记录视频

	public static boolean isIntentAvailable(Context context, String action) {
		final PackageManager packageManager = context.getPackageManager();
		final Intent intent = new Intent(action);
		List<ResolveInfo> list = packageManager.queryIntentActivities(intent, PackageManager.MATCH_DEFAULT_ONLY);
		return list.size()>0;
	}

	private void dispatchTakeVideoIntent() {
		Intent takeVideoIntent = new Intent(MediaStore.ACTION_VIDEO_CAPTURE);
		startActivityForResult(takeVideoIntent, ACTION_TAKE_VIDEO);
	}

###观看视频

	private void handleSmallCameraVideo(Intent intent) {
		mVideoUri = intent.getData();
		mVideoView.setVideoURI(mVideoUri);

##控制摄像头

###打开摄像头

在实际应用中，经常在 android.app.Activity.onResume() 中获取 android.hardware.Camera 的 instance。

	private boolean safeCameraOpen(int id) {
		boolean qOpened = false;
		try {
			releaseCameraAndPreview();
			mCamera = Camera.open(id);
			qOpened = (mCamera != null);
		} catch (Exception e) {
			...
		}

		return qOpened;
	}

	private void releaseCameraAndPreview() {
		mPreview.setCamera(null);
		if (mCamera != null) {
			mCamera.release();
			mCamera = null;
		}
	}

###使用摄像头预览

开发者可以使用 android.view.SurfaceView class 获取摄像头的预览图像。

为了使用预览，开发者首先需要实现 android.view.SurfaceHolder.Callback 接口，这个接口用来将图像从摄像头传输到应用程序。

	class Preview extends ViewGroup implements SurfaceHolder.Callback {
		SurfaceView mSurfaceView;
		SurfaceHolder mHolder;

		Preview(Context context) {
			super(context);
			mSurfaceView = new SurfaceView(context);
			addView(mSurfaceView);

			mHolder = mSurfaceView.getHolder();
			mHolder.addCallback(this);
			mHolder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
		}
		...
	}

在获取到 android.view.SurfaceView 的 instance 以后，开发者可以通过调用 android.hardware.Camera.startPreview() method 来启动预览

	public void setCamera(Camera camera) {
		if (mCamera == camera) {
			return;
		}
	
		stopPreviewAndFreeCamera();
		mCamera = camera;
	
		if (mCamera != null) {
			List<Size> localSizes = mCamera.getParameters().getSupportedPreviewSizes();
			mSupportedPreviewSizes = localSizes;
			requestLayout();
	
			try {
				mCamera.setPreviewDisplay(mHolder);
			} catch (IOException e) {
				e.printStackTrace();
			}
	
			mCamera.startPreview();
		}
	}

###更改摄像头的设置

开发者可以调用 android.hardware.Camera.setParameters() 来更改摄像头的设置：

	public void surfaceChanged(SurfaceHolder holder, int format, int w, int h) {
		Camera.Parameters parameters = mCamera.getParameters();
		parameters.setPreviewSize(mPreviewSize.width, mPreviewSize.height);
		requestLayout();
		mCamera.setParameters(parameters);
		mCamera.startPreview();
	}

###拍照片

开发者可以使用 android.hardware.Camera.takePicture() method 从预览画面拍照片。开发者可以创建一个 android.hardware.Camera.PictureCallback 和 android.hardware.Camera.ShutterCallback objects，然后传递给 android.hardware.Camera.takePicture()。

如果开发者需要连拍，可以使用一个实现了 android.hardware.Camera.PreviewCallback.onPreviewFrame() 接口 android.hardware.Camera.PreviewCallback class。

###重启预览

当拍了照片时，开发者必须手动重启摄像头预览

	public void onClick(View v) {
		switch (mPreviewState) {
			case K_STATE_FROZEN:
				mCamera.startPreview();
				mPreviewState = K_STATE_PREVIEW;
				break;
			default:
				mCamera.takePicture(null, rawCallback, null);
				mPreviewState = K_STATE_BUSY;
		}
		shutterBtnConfig();
	}

###停止预览

	public void surfaceDestroyed(SurfaceHolder holder) {
		if (mCamera != null) {
			mCamera.stopPreview();
		}
	}

	private void stopPreviewAndFreeCamera() {
		if (mCamera != null) {
			mCamera.stopPreview();
			mCamera.release();
			mCamera = null;
		}
	}
