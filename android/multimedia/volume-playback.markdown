#控制音频

##控制音量和播放

###选择音频 stream

android 系统有一个独立的音频 stream，可以用来播放音乐、闹铃等等。

###使用音量按键控制音量

默认情况下，音量按键可以控制活跃在系统的音频 stream。如果开发者的程序没有使用音频 stream，那么音量按键可以调节铃音音量。

如果开发者需要通过音量按键来调节程序的音频 stream 的音量，可以使用 android.app.Activity.setVolumeControlStream() 将音量按键事件重定向到音频 stream。

开发者选择合适的音频 stream 以后，需要将它设置为活跃的音频 stream，一般应该在 android.app.Activity.onCreate() 中完成。

	setVolumeControlStream(AudioManager.STREAM_MUSIC);

###使用播放控制键控制音频播放

当用户使用硬件上的音频播放控制按键的时候，系统会 boradcast 一个附带 android.content.Intent.ACTION\_MEDIA\_BUTTON action 的 android.content.Intent。为了响应这个 android.content.Intent，开发者需要在 manifest 文件里注册 android.content.BroadcastReceiver 监听器。

	<receiver android:name=".RemoteControlReceiver">
		<intent-filter>
			<action android:name="android.intent.action.MEDIA_BUTTON" />
		</intent-filter>
	</receiver>

这个 android.content.Intent 包含按键的具体信息

	public class RemoveControlReceiver extends BroadcastReceiver {
		public void onReceive(Context context, Intent intent) {
			if (Intent.ACTION_MEDIA_BUTTON.equals(intent.getAction())) {
				KeyEvent event = (KeyEvent) intent.getParcelableExtra(Intent.EXTRA_KEY_EVENT);
				if (KeyEvent.KEYCODE_MEDIA_PLAY == event.getKeyCode()) {
					...
				}
			}
		}
	}

开发者可以使用 android.media.AudioManager 选择监听或不监听音频按键事件

	AudioManager am = mContext.getSystemService(Context.AUDIO_SERVICE);
	am.registerMediaButtonEventReceiver(RemoteControlReceiver);
	am.unregisterMediaButtonEventReceiver(RemoteControlReceiver);

一般来说，当程序不再运行的时候，需要取消监听音频按键事件。

##管理音频焦点

android 系统使用音频焦点来管理正在播放的音频。当开发者的程序需要播放音频的时候，一般需要请求获取音频焦点，并且在播放的过程中能够响应失去焦点的事件。

###请求音频焦点

当引用程序需要播放音频的时候，一般需要调用 android.media.AudioManager.requestAudioForcus() method 来请求获得焦点，这个 method 一般会返回 android.media.AudioManager.AUDIOFOCUS\_REQUEST\_GRANTED。

开发者必须指定需要播放的音频 stream，也必须指定请求短时焦点还是长时焦点。

下面的代码获取长时焦点：

	AudioManager am = mContext.getSystemService(Context.AUDIO_SERVICE);

	int result = am.requestAudioForcus(afChangeListener, AudioManager.STREAM_MUSIC, AudioManager.AUDIOFOCUS_GAIN);
	if (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
		am.unregisterMediaButtonEventReceiver(RemoteControlReceiver);
	}

播放完音频以后，开发者需要调用 android.media.AudioManager.abandonAudioFocus() method。

	am.abandonAudioFocus(afChangeListener);

当播放短时音频时，开发者可以指定播放的音频接受不接收 ducking 模式。在这种模式下，需要正在播放音频的程序调低一下音量。

	int result = am.requestAudioForcus(afChangeListener, AudioManager.STREAM_MUSIC, AudioManager.AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK);
	if (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
		...
	}

###处理短时音频失焦

android.media.AudioManager.onAudioFocusChange() 回调函数可以用来处理音频失焦。有三种情况：

- 长时失焦，程序应该停止播放音频，移出音量按键的 listener，抛弃音频焦点。
- 短时失焦，程序应该静音，并继续监听音频焦点事件，一旦焦点回来，继续播放音频。
- 短时 ducking 失焦。

代码：

	OnAudioFocusChangeListener afChangeListener = new OnAudioFocusChangeListener() {
		public void onAudioFocusChange(int focusChange) {
			if (focusChange == AUDIOFOCUS_LOSS_TRANSIENT) {
				// Pause playback
			} else if (focusChange == AudioManager.AUDIOFOCUS_GAIN) {
				// Resume playback
			} else if (focusChange == AudioManager.AUDIOFOCUS_LOSS) {
				am.unregisterMediaButtonEventReceiver(RemoteControlReceiver);
				am.abandonAudioFocus(afChangeListener);
			}
		}
	}

###duck 模式

duck 模式是为其他程序的音频播放事件调低音量

	OnAudioFocusChangeListener afChangeListener = new OnAudioFocusChangeListener() {
		public void onAudioFocusChange(int focusChange) {
			if (focusChange == AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK) {
				// Lower the volume
			} else if (focusChange == AudioManager.AUDIOFOCUS_GAIN) {
				// Raise it back to normal
			}
		}
	}

##管理音频输出设备

###检查设备是否在使用中

开发者可以通过查询 android.media.AudioManager 来检测当前的音频输出设备是喇叭、耳机还是蓝牙设备

	if (isBluetoothA2dpOn()) {
	} else if (isSpeakerphoneOn()) {
	} else if (isWiredHeadsetOn()) {
	} else {
	}

###处理音频输出设备改变事件

当音频输出设备从喇叭切换回耳机的时候，系统会 boradcast 一个 android.media.AudioManager.ACTION\_AUDIO\_BECOMING\_NOISY android.content.Intent

	private class NoisyAudioStreamReceiver extends BroadcastReceiver {
		public void onReceive(Context context, Intent intent) {
			if (AudioManager.ACTION_AUDIO_BECOMING_NOISY.equals(intent.getAction())) {
				// Pause the playback
			}
		}
	}

	private IntentFilter intentFilter = new IntentFilter(AudioManager.ACTION_AUDIO_BECOMING_NOISY);

	private void startPlayback() {
		registerReceiver(myNoisyAudioStreamReceiver(), intentFilter);
	}

	private void stopPlayback() {
		unregisterReceiver(myNoisyAudioStreamReceiver);
	}
