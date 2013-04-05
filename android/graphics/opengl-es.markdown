#使用 OpenGL ES

##配置 OpenGL ES 环境

要使用 OpenGL，需要实现 android.opengl.GLSurfaceView 以及 android.opengl.GLSurfaceView.Renderer，前者是展示 OpenGL 图像的 view container，后者是 OpenGL 渲染工具。

###在 manifest 文件中配置 OpenGL ES

如果需要使用 OpenGL ES 2.0 API，需要增加以下内容：

	<uses-feature android:glEsVesion="0x00020000" android:required="true" />

如果需要使用 texture 压缩功能，则必须添加如下内容

	<support-gl-texture android:name="GL_OES_compressed_ETC1_RGB8_texture" />
	<support-gl-texture android:name="GL_OES_compressed_ETC1_paletted_texture" />

###创建一个使用 OpenGL ES 的 android.app.Activity

	public class OpenGLES20Activity extends Activity {
		private GLSurfaceView mGLView;

		public void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);

			mGLView = new MyGLSurfaceView(this);
			setContentView(mGLView);
		}
	}

###创建 android.opengl.GLSurfaceView class

一个 android.opengl.GLSurfaceView object 是一个用来画 OpenGL ES 图形的 view。

	class MyGLSurfaceView extends GLSurfaceView {
		public MyGLSurfaceView(Context context) {
			super(context);
			setRenderer(new MyRenderer());
			// for OpenGL ES 2.0
			setEGLContextClientVersion(2);
			setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY);
		}
	}

###创建一个 android.opengl.GLSurfaceView.Renderer class

这个 class 决定着当 android.opengl.GLSurfaceView 里的图形变化时，显示什么。它最重要的三个 methods 是：

- android.opengl.onSurfaceCreated()，只需要调用一次，配置 OpenGL ES 环境
- android.opengl.onDrawFrame()，每次重绘图形的时候都会调用
- android.opengl.onSurfaceChanged()，每当 view 的几何形状改变都会被调用

一个最简单的 android.opengl.GLSurfaceView.Renderer 实现：

	public class MyRenderer implements GLSurfaceView.Renderer {
		public void onSurfaceCreated(GL10 unused, EGLConfig config) {
			GLES20.glClearColor(0.5f, 0.5f, 0.5f, 1.0f);
		}

		public void onDrawFrame(GL10 unused) {
			GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT);
		}

		public void onSurfaceChanged(GL10 unused, int width, int height) {
			GLES20.glViewport(0,0,width,height);
		}
	}

##定义形状

OpenGL ES 支持三维空间坐标

###定义一个三角形

在定义三角形的时候，必须定义坐标。典型的方式定义一个顶点数组

	class Triangle {
		private FloatBuffer vertexBuffer;

		static final int COORDS_PER_VERTEX = 3;
		// in counterclockwise order
		static float triangleCoords[] = {
			 0.0f,  0.622f,  0.0f,
			-0.5f, -0.311f,  0.0f,
			 0.5f, -0.311f,  0.0f
		};

		float color[] = { 0.6367, 0.7695, 0.2227, 1.0f};

		public Triangle() {
			ByteBuffer bb = ByteBuffer.allocateDirect(triangleCoords.length * 4);
			bb.order(ByteOrder.nativeOrder());
			vertexBuffer = bb.asFloatBuffer();
			vertexBuffer.put(triangleCoords);
			vertexBuffer.position(0);
		}
	}

默认情况下，OpenGL ES 认为 android.opengl.GLSurfaceView 的中心坐标为 [0,0,0]\(X,Y,Z)，右上角是 [1,1,0]，左下角是 [-1,-1,0]，

###定义一个方形

	class Square {
		private FloatBuffer vertexBuffer;
		private ShortBuffer drawListBuffer;

		static final int COORDS_PER_VERTEX = 3;
		static float squareCoords[] = {
			-0.5f,  0.5f,  0.0f,
			-0.5f, -0.5f,  0.0f,
			 0.5f, -0.5f,  0.0f,
			 0.5f,  0.5f,  0.0f
		}

		private short drawOrder[] = { 0,1,2,0,2,3 };

		public Square() {
			ByteBuffer bb = ByteBuffer.allocateDirect(squareCoords.length * 4);
			bb.order(ByteOrder.nativeOrder);
			vertexBuffer = bb.asFloatBuffer();
			vertexBuffer.put(squareCoords);
			vertexBuffer.position(0);

			ByteBuffer dlb = ByteBuffer.allocateDirect(drawOrder.length * 2);
			dlb.order(ByteOrder.nativeOrder);
			drawListBuffer = dlb.asShortBuffer();
			drawListBuffer.put(drawOrder);
			drawListBuffer.position(0);
		}
	}

##绘图

###初始化图形

除非图形的坐标会有变化，一般是在 android.opengl.GLSurfaceView.Renderer.onSurfaceCreated() method 里初始化

	public void onSurfaceCreated(GL10 unused, EGLConfig config) {
		mTriangle = new Triangle();
		mSquare = new Square();
	}

###绘图

在 OpenGL ES 2.0 里绘图，需要:

- Vertex Shader，用来绘画顶点
- Fragment Shader，用来绘画表面，包括颜色和 texture
- Program，一个包含 Shaders 的 OpenGL ES object 

开发者至少需要一个 Vertex Shader 和一个 Fragment Shader

	private final String vertexShaderCode =
		"attribute vec4 vPosition;" +
		"void main() {"             +
		"	gl_Position = vPosition;" +
		"}";

	private final String fragmentShaderCode =
		"precision mediump float;"  +
		"uniform vec4 vColor;"      +
		"void main() {"             +
		"	gl_FragColor = vColor;"   +
		"}";

Shaders 包含有 OpenGL Shading Language，必须在使用前编译。

	public static int loadShader(int type, String shaderCode) {
		int shader = GLES20.glCreateShader(type);
		GLES20.glShaderSource(shader, shaderCode);
		GLES20.glCompileShader(shader);

		return shader;
	}

编译 OpenGL ES shaders 需要耗费较大的 CPU 资源，开发者最好不要运行多次编译过程

	public Triangle() {
		int vertexShader = loadShader(GLES20.GL_VERTEX_SHADER, vertexShader);
		int fragmentShader = loadShader(GLES20.GL_FRAGMENT_SHADER, fragmentShaderCode);

		mProgram = GLES20.glCreateProgram()
		GLES20.glAttachShader(mProgram, vertexShader);
		GLES20.glAttachShader(mProgram, fragmentShader);
		GLES20.glLinkProgram(mProgram);
	}

图形 class 里还需要定义 draw() method：

	public void draw() {
		GLES20.glUseProgram(mProgram);
		mPositionHandle = GLES20.glGetAttribLocation(mProgram, "vPosition");
		GLES20.glEnableVertexAttribArray(mPositionHandle);
		GLES20.glVertexAttribPointer(mPositionHandle, COORDS_PER_VERTEX, GLES20.GL_FLOAT, false, vertexStride, vertexBuffer);
		mColorHandle = GLES20.glGetUniformLocatoin(mProgram, "vColor");
		GLES20.glUniform4fv(mColorHandle, 1, color, 0);
		GLES20.glDrawArrays(GLES20.GL_TRIANGLE, 0, vertexCount);
		GLES20.glDisableVertexAttribArray(mPositionHandle);
	}

只需要在图形的渲染器 class 的 android.opengl.GLSurfaceView.Renderer.onDrawFrame() method 里调用 draw() method 就行了。

##使用 projection 和 camera views

projection 和 camera views 可以使 OpenGL ES 画出的图形不随屏幕的变化而变化。

###定义 projection

	public void onSurfaceChanged(GL10 unused, int width, int height) {
		GLES20.glViewport(0, 0, width, height);
		float ration = (float) width/height;
		Matrix.frustumM(mProjMatrix, 0, -ration, ration, -1, 1, 3, 7);
	}

###定义 camera views

	public void onDrawFrame(GL10 unused) {
		Matrix.setLookAtM(mVMatrix, 0, 0, 0, -3, 0f, 0f, 0f, 0f, 1.0f, 0.0f);
		Matrix.multiplyMM(mMVPMatrix, 0, mProjMatrix, 0, mVMatrix, 0);
		mTriangle.draw(mMVPMatrix);
	}

###使用 projection 和 camera views

	public void draw(float[] mvpMatrix) {
		mMVPMatrixHandle = GLES20.glGetUniformLocatoin(mProgram, "mMVPMatrix");
		GLES20.glUniformMatrix4v(mMVPMatrixHandle, 1, false, mvpMatrix, 0);
		GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, vertexCount);
	}

###增加运动效果

相比于 android.graphics.Canvas 和 android.graphics.drawable.Drawable classes，OpenGL ES 提供运动的效果。

###旋转图形

开发者可以创建一个变换矩阵：

	private float[] mRotationMatrix = new float[16];

	public void onDrawFrame(GL10 gl) {
		...

		long time = SystemClock.uptimeMillis() % 4000L;
		float angle = 0.090f * ((int)time);
		Matrix.setRotateM(mRotationMatrix, 0, mAngle, 0, 0, -1.0f);
		Matrix.multiplyMM(mMVPMatrix, 0, mRotationMatrix, 0, mMVPMatrix, 0);
		mTriangle.draw(mMVPMatrix);
	}

###持续渲染

	public MyGLSurfaceView(Context context) {
		// setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY);
	}

##响应触摸事件

###配置一个触摸监听器

	public boolean onTouchEvent(MotionEvent e) {
		float x = e.getX();
		float y = e.getY();

		switch (e.getAction()) {
			case MotionEvent.ACTION_MOVE:
				float dx = x - mPreviousX;
				float dy = y - mPreviousY;

				if (y > getHeight()/2) {
					dx = dx * -1;
				}

				if (x < getWidth() / 2) {
					dy = dy * -1;
				}

				mRenderer.mAngle += (dx + dy) * TOUCH_SCALE_FACTOR;
				requestRender();
		}

		mPreviousX = x;
		mPreviousY = y;

		return true;
	}

###应用旋转

	public void onDrawFrame(GL10 gl) {
		Matrix.setRotateM(mRotationMatrix, 0, mAngle, 0, 0, -1.0f);
		Matrix.multiplyMM(mMVPMatrix, 0, mRotationMatrix, 0, mMVPMatrix, 0);
		mTriangle.draw(mMVPMatrix);
	}
