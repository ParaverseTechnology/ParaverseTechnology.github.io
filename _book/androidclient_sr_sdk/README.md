# 云雀安卓客户端SDK V3.1.7.0

## 说明

### 1. 内容

云雀安卓客户端 SDK 包括如下内容：

辅助获取应用列表，进入应用（配置多人互动模式等.创建与云端服务器的连接解码并显示画面，发送键盘鼠标指令等。

### 2.版本号

云雀安卓客户端 SDK 版本号与云雀服务器主版本号对应(即前两位)。应使用相同版本的客户端和服务器。

### 3.更新

云雀安卓客户端 SDK 通过打包的 aar 文件引入依赖。通过更换 aar 文件更新。

> 更新 aar 文件后需清理工程项目。

## 使用

> 以下示例代码使用 gradle-4.6。其他版本请根据具体情况调节。

### 1. 依赖平台

* 云雀 sdk 中 minSdkVersion 为 19.

* webrtc 依赖的 Android SDK 版本 compileSdkVersion 应不小于 25.

> 推荐总是使用最新的 SDK 进行编译。

* 依赖 Java Version 为 1.8

```gradle
compileOptions {
       sourceCompatibility JavaVersion.VERSION_1_8
       targetCompatibility JavaVersion.VERSION_1_8
}
```

### 2.引用资源

在项目中引用 lark-sr-kit-*-build-*.aar 。

build.gradel 的 android 字段下添加.

```gradle
repositories {
       flatDir {
              dirs 'libs'
       }
}
```

将 lark-sr-kit-3.1.7.0-build-1.aar 和 libwebrtc_20200816.aar 复制到项目libs目录下。 dependencies 内添加:

```gradle
// webrtc
// 3.1.2.5 更改
implementation(name: 'libwebrtc_20200925_M79', ext: 'aar')
// larksr
implementation(name: 'lark-sr-kit-3.1.7.0-build-1', ext: 'aar')
```

> 从 2.10.5.0 起使用单独编译的 webrtc 库，不再使用公开发布的 webrtc 库。应使用本 sdk 中附带的版本。

### 3.依赖库

dependencies 内添加:

```gradle
// 3.1.2.0 升级为 3.12.+
implementation 'com.squareup.okhttp3:okhttp:3.12.+'
// 3.1.2.0 新增, 可选
implementation 'com.github.bumptech.glide:glide:4.11.0'
```

### 4.所需权限

mainfist 中权限将从 larksr sdk 中合并到一起。

### 5.更新

将 lark-sr-kit-3.1.7.0-build-1.aar 和 libwebrtc_20200529.aar 复制到项目libs目录下。
注意修改 gradle 文件引用最新的sdk:

```gradle
// webrtc
implementation(name: 'libwebrtc_20200529', ext: 'aar')
// larksr
implementation(name: 'lark-sr-kit-3.1.7.0-build-1', ext: 'aar')
```

## 调用

### 初始化

使用 SDK 之前应调用初始化接口。调用 CloudlarkManager 的静态方法。

### SDK授权校验

3.1.3.0 新增授权校验接口，当 SDK 初始化之后应调用授权校验接口验证授权 ID。初始化授权 ID 失败将抛出异常。
如果没有获取授权 ID 应联系商务取得。
APP 首次启动将连接网络进行授权验证。授权成功之后在授权有效期内可不连接外网。

```java
public static boolean initSdkAuthorization(Context context, String sdkId) throws LarkSdkAuthorizationException
```

SDK 初始化流程示例：

```java
CloudlarkManager.init(this, CloudlarkManager.APP_TYPE_SR);

String sdkId = "您的SDK ID. 如果没有请联系商务获取。";
try {
    CloudlarkManager.initSdkAuthorization(this, sdkId);
    Log.d(TAG, "lark sdk auth success");
} catch (CloudlarkManager.LarkSdkAuthorizationException e) {
    Log.d(TAG, "lark sdk auth failed." + e.getLocalizedMessage() + " " + e.getLarkEventCode());
    toastInner("Lark 授权失败 " + e.getLocalizedMessage());
    e.printStackTrace();
}
```

#### 手动设置客户端凭证

```java
/**
 * 初始化
 * @param context Android 上下文
 * @param type 系统类型
 * @param appKey 客户端凭证，appkey，从云雀服务器后台获取
 * @param appSecret 客户端凭证密钥，从云雀后台获取
 */
public static void init(Context context, String type, String appKey, String appSecret)
```

传入的 appKey 和 appSecret 将用于调用后台接口时验证客户端凭证。客户端凭证应从服务器管理员处获取。

传入的 appKey 和 appSecret 将保存在应用目录下 certificate_appkey.txt 和  certificate_appsecret.txt 文件中。初始化时如果这两个文件不存在将创建。

> 目录为 /data/data/PACKAGE_NAME/files/cloudlark_setup/  如果又存储卡将保存在存储卡中，否则保存在内部文件中。

#### 自动设置客户端凭证

如果选择不传入 appkey 和 appsecret 进行初始化，将从上面的文件中读取 appkey 和 appsecret。

```java
/**
 * 初始化
 * @param context Android 上下文
 */
public static void init(Context context)
```

### 获取应用列表

调用后台接口之前应先设置后台服务器的地址。调用静态方法 `Base.setServerAddr(mServerIp);` 进行设置。更换服务器地址应及时设置。

以下为调用的伪代码，具体使用请参考 demo

```java
Base.setServerAddr(mServerIp);
```

调用  GetAppliList 类的 getAppliList/getAppliListSync 成员方法将调用服务端接口，获取应用列表。应用列表数据将通过回调函数返回。

> 可使用  ScheduleTaskManager 开启任务循环，使用方式见 demo。

以下为调用的伪代码，具体使用请参考 demo

```java

GetAppliList getAppliList = new GetAppliList(new GetAppliList.Callback() {
    @Override
    public void onSuccess(List<AppListItem> list) {}

    @Override
    public void onFail(String s) {}
});
getAppliList.getAppliListSync();
```

### 进入应用

#### 普通模式调用如下

调用  EnterAppliInfo 类的 enterApp 成员方法将调用进入应用接口，结果通过回调函数返回。

以下为调用的伪代码，具体使用请参考 demo

```java
private EnterAppliInfo.Callback mEnterAppliInfoCallback = new EnterAppliInfo.Callback() {
       @Override
       public void onSuccess(RtcClient.Config rtcParams) {
              Intent intent = new Intent();
              ComponentName componentName = new ComponentName(MainActivity.this, RtcActivity.class);
              intent.setComponent(componentName);
              // 进入课程页面。
              intent.putExtra(RtcClient.Config.name, rtcParams);
              MainActivity.this.startActivity(intent);
       }

       @Override
       public void onFail(String err) {
           toastInner(err);
       }
};

// face enter appli
enterAppliInfo.enterApp(item);
```

#### 互动模式参考《云雀安卓客户端SDK更新说明》

### RtcClient 类用于创建和管理与云雀服务端的连接

#### 构造函数

```java
/**
* RtcClient 客户端。
* */
RtcClient(Config config, VideoSink render, RtcClientEvent callback, Activity activity)
```

* config 为所需参数。具体设置如下。

> 可从云雀后台服务 getEnterAppliInfo 接口取得。取得方法参见 demo。

```java
// task id
public String taskId = "";
// 服务器地址
public String appServer = "";
// 服务器端口号
public int appPort = 0;
// 优先外网 ip
public String preferPubOutIp = "";
// 码率
public int bitrateKbps = 5 * 1000;
// 帧率
public int fps = 30;
// 无操作超时
public int noOperationTimeout = 0;
// 是否使用代理
public int wsProxy = 0;
// 是否使用安全协议
public boolean useSecurityProtocol = false;
// web server 地址
public String webServerIp = "";
// web server 端口号
public int webServerPort = 0;
public int wm = 0;
// 互动模式相关
public int playerMode = 0;
// 互动模式下用户类型
public int userType = 1;
// 互动模式下用户昵称
public String nickname = "";
// 互动模式下的房间码
public String roomCode = "";
// 背景颜色（rtcclient外部使用）
public String bgColor = "";
// 是否使用手柄
public boolean useGamepad = false;
```

* render 为渲染控件。

#### 方法

1. 回调函数

```java
/**
 * socket 连接成功时回调
 */
void onConnect();
/**
 * 登录服务器成功
 */
void onLoginSuccess(int uid);
/**
 * 首次收到视频帧时回调
 */
void onMediaReady();

/**
 * 视频帧大小变化时回调
 */
void onFrameResolutionChanged(int videoWidth, int videoHeight, int rotation);
/**
 * socket 断开连接
 */
void onDisconnect();
/**
 * 无操作超时
 */
void onNoOpreationTimeout();
/**
 * 显示信息
 * @param msg 描述
 */
void onInfo(String msg);

/**
 * 出现错误时
 * @param err 错误描述
 */
void onError(String err);

/**
 * 云端应用大小变化时
 */
void onAppResize(AppNotification.AppResize mouseLockRect);

/**
 * 云端应用鼠标状态变化时回调
 */
void onMouseState(AppNotification.AppMouseMode mouseMode);

/**
 * 玩家列表
 */
void onPlayerList(List<AppNotification.PlayerDesc> playerList);
/**
 * 视频连接状态报告
 */
void onPeerStatusReport(SampleRTCStats sampleRTCStats);
/**
 *  数据通道打开
 */
void onDataChannelOpen();
/**
 *  数据通道关闭
 */
void onDataChannelClose();
/**
 * 数据通道字符消息
 */
void onDataChannelMessage(String msg);
/**
  * 数据通道字节消息
  */
void onDataChannelMessage(byte[] buffer);
/**
 * 应用请求输入字符
 */
void onAppRequestInput(boolean enable);
/**
 * 应用请求手柄震动
 */
void onAppRequestGamepadOutput();
```

2. 互动模式相关

```java
/**
* 是否时观看者
* @return 是否是观看者
*/
public boolean isObserver()
```

```java
/**
* 切换操作权限，只有房主才有权限操作。
* @return 是否成功
*/
public boolean dispatchController(int uid)
```

3. 操作相关

鼠标键盘接口统一变更如下. 其中需要注意的时按键按下时的 isRepeat 参数，当统一按键重复触发按下时 isRepeat 应为 true。

```java
/**
* 发送鼠标移动。
*/
public boolean sendMouseMove(int x, int y, int rx, int ry)
// 新增
public boolean sendMouseMove(ClientInput.MouseMove mouseMove)

/**
* 发送鼠标按下。
*/
public boolean sendMouseDown(int x, int y, ClientInput.MouseKey key)
// 新增
public boolean sendMouseDown(ClientInput.MouseDown mouseDown)

/**
* 发送鼠标抬起。
*/
public boolean sendMouseUp(int x, int y, ClientInput.MouseKey key)
// 新增
public boolean sendMouseUp(ClientInput.MouseUp mouseUp)

/**
* 发送鼠标滚轮。
*/
public boolean sendMouseWheel(int x, int y, int det)
// 新增
public boolean sendMouseWheel(ClientInput.MouseWheel mouseWheel)

/**
* 发送键盘按下。
*/
public boolean sendKeyDown(int vkey, boolean isRepeat)
// 新增
public boolean sendKeyDown(ClientInput.KeyDown keyDown)

/**
* 发送键盘抬起。
*/
public boolean sendKeyUp(int vkey)
// 新增
public boolean sendKeyUp(ClientInput.KeyUp keyUp)
```

### 手柄相关接口

手柄接口消息按照 windows xbox 360 手柄标准定义，即包含 xbox 360 手柄的功能，如按钮，摇杆，扳机键。windows 上最多支持4个手柄，当前版本服务端只处理1个手柄，后续会放开多个手柄的支持。

接口中发送的按键值为对应的 windows 中定义的按键值。类 *WindowsXInputGamepad* 定义了这些值。可用直接对应使用。

#### 直接调用接口

* 手柄按键按下

```java
/**
* 手柄按键按下
* @param userIndex 硬件设备索引 0-3
* @param key 按键按下码 @see WindowsXInputGamepad
* @param isRepeat 是否重复按下
* @return 是否发送成功
*/
public boolean sendGamepadButtonDown(int userIndex, int key, boolean isRepeat)
/**
* 手柄按键按下
* @param buttonDown @see ClientInput.GamepadInputButtonDown
* @return 是否发送成功
*/
public boolean sendGamepadButtonDown(ClientInput.GamepadInputButtonDown buttonDown)
```

* 手柄按键抬起

```java
/**
* 手柄按键抬起
* @param userIndex userIndex 硬件设备索引 0-3
* @param key @see WindowsXInputGamepad
* @return
*/
public boolean sendGamepadButtonUp(int userIndex, int key)
/**
* 手柄按键抬起
* @param buttonUp @see ClientInput.GamepadInputButtonUp
* @return 是否发送成功
*/
public boolean sendGamepadButtonUp(ClientInput.GamepadInputButtonUp buttonUp)
```

* 发送摇杆状态

```java
/**
* 发送摇杆状态。同时发送左摇杆和右摇杆状态
* @param stickStates @see ClientInput.GamepadInputJoyStickStates 摇杆值为 -32767 到 32767
* @return 是否发送成功
*/
public boolean sendGamepadJoyStickStates(ClientInput.GamepadInputJoyStickStates stickStates)
/**
    * 发送扳机键状态
    * @param index userIndex userIndex 硬件设备索引 0-3
    * @param isLeft 是否是左扳机键
    * @param value 扳机键的值 0-255
    * @return 是否发送成功
    */
public boolean sendGamepadTrigger(int index, boolean isLeft, int value)
```

其中 GamepadInputJoyStickStates 主要定义以下字段

```java
// win     MAX
//          |
// MIN-------------MAX
//         |
//        MIN
public static class GamepadInputJoyStickStates {
    // 硬件序号
    private int userIndex;
    // 左摇杆 X 值，-32767 到 32767
    private int thumbLX;
    // 左摇杆 Y 值，-32767 到 32767
    private int thumbLY;
    // 右摇杆 X 值，-32767 到 32767
    private int thumbRX;
    // 右摇杆 Y 值，-32767 到 32767
    private int thumbRY;
```

* 发送扳机状态

```java
/**
* 发送扳机键状态
* @param trigger @see ClientInput.GamepadInputTriger
* @return 是否发送成功
*/
public boolean sendGamepadTrigger(ClientInput.GamepadInputTriger trigger)
/**
* @param keyUp 向云端应用发送键盘事件。
*/
public boolean sendKeyUp(ClientInput.KeyUp keyUp)
```

#### 预处理封装

可使用 *WindowsXInputGamepad* 类定义好的预处理函数直接发送手柄消息给云端

* 处理摇杆等事件

```java
public static boolean processGenericMotionEvent(MotionEvent motionEvent, RtcClient rtcClient) throws IllegalArgumentException
```

* 处理按钮事件

```java
public static ClientInput.GamepadInputJoyStickStates processJoystickInput(MotionEvent event, int historyPos)
```

##### 手柄消息中需要的一般常量

手柄消息中发送的按键码等为 Windows 定义的 xbox360 手柄的标准码。在 *WindowsXInputGamepad* 类有相关定义：

```java
// windows 定义的摇杆最大值
public static int JOYSTICK_AXIS_MAX = 32767;
// windows 定义的摇杆最小值
public static int JOYSTICK_AXIS_MIN = -32767;
// windows 定义的扳机键最大值
public static int JOYSTICK_TRIGGER_MAX = 255;
// windows 定义的班级键最小值
public static int JOYSTICK_TRIGGER_MIN = 0;
// windows DPAD UP 按键码
public static int XINPUT_GAMEPAD_DPAD_UP        = 0x0001;
// windows DPAD DOWN 按键码
public static int XINPUT_GAMEPAD_DPAD_DOWN      = 0x0002;
// windows DPAD LEFT 按键码
public static int XINPUT_GAMEPAD_DPAD_LEFT      = 0x0004;
// windows DPAD RIGHT 按键码
public static int XINPUT_GAMEPAD_DPAD_RIGHT     = 0x0008;
// windows 手柄 START 键按键码
public static int XINPUT_GAMEPAD_START          = 0x0010;
// windows 手柄 BACK/SELECT 键按键码
public static int XINPUT_GAMEPAD_BACK           = 0x0020;
// windows 手柄 LeftThumb 即左摇杆按下键按键码
public static int XINPUT_GAMEPAD_LEFT_THUMB     = 0x0040;
// windows 手柄 RightThumb 即右摇杆按下键按键码
public static int XINPUT_GAMEPAD_RIGHT_THUMB    = 0x0080;
// windows 手柄 LeftShoulder 左肩按键码
public static int XINPUT_GAMEPAD_LEFT_SHOULDER  = 0x0100;
// windwos 手柄 RightShoulder 右肩按键码
public static int XINPUT_GAMEPAD_RIGHT_SHOULDER = 0x0200;
// windows 手柄 gamepad a 按键
public static int XINPUT_GAMEPAD_A              = 0x1000;
// windows 手柄 gamepad b 按键
public static int XINPUT_GAMEPAD_B              = 0x2000;
// windows 手柄 gamepad x 按键
public static int XINPUT_GAMEPAD_X              = 0x4000;
// windows 手柄 gamepad y 按键
public static int XINPUT_GAMEPAD_Y              = 0x8000;
```

可使用 *WindowsXInputGamepad* 中定义的以下静态函数转换 Android 手柄常量到所需要的 Windows 手柄常量.

```java
/**
* android 按键
* @param keyCode android keycode
* @return windows 对应的手柄按键,转换失败返回 XINPUT_UNKNOWN
*/
public static int getWinXinputCodeFromAndroidKeyCode(int keyCode)
/**
* 转换 Android 的摇杆值到 windows 摇杆值。
* @param axis JOYSTICK_AXIS_Y/JOYSTICK_AXIS_X
* @param value android value
* @return windows value
*/
public static int getWinThumbValueFromAndroidJoystickAxis(int axis, float value)

/**
* android 按键
* @param keyCode android keycode
* @return windows 对应的手柄按键,转换失败返回 XINPUT_UNKNOWN
*/
public static int getWinXinputCodeFromAndroidKeyCode(int keyCode)
```

#### 输入文字

当云端应用调取输入法时将触发回调函数, 当 enable 为 true 时开启输入法，false 时关闭输入法。

```java
/**
 * 应用请求输入字符
 */
void onAppRequestInput(boolean enable);
```

当开启输入法时，应在本地打开文字输入框让或让用户选择，让用户输入。通过 RtcClient 的: 

```java
/**
 * 输入文本给云端应用
 * 需要当云端应用输入组件获取到焦点时发送
 * 云端应用输入组件失去焦点时发送不会成功
 * @param txt
 * @return
 */
public void sendInputText(String txt)
```

方法将用户输入的文字发送给云端应用。

#### DataChannel 功能

云端应用集成云雀数据通道功能并在后台成功设置之后，可使用数据通道相关接口，发送文本或字节消息给云端应用。相关回调函数将被调用.

```java
/**
 *  数据通道打开
 */
void onDataChannelOpen();
/**
 *  数据通道关闭
 */
void onDataChannelClose();
/**
 * 数据通道字符消息
 */
void onDataChannelMessage(String msg);
/**
 * 数据通道字节消息
 */
void onDataChannelMessage(byte[] buffer);
```

数据通道开启后可向云端应用发送数据

```java
/**
 * 发送字符消息给云端应用数据通道
 * @param msg
 * @return 是否发送成功
 */
public boolean sendToAppDataChannel(String msg)
/**
 * 发送字节消息给云端应用数据通道
 * @param data
 * @return 是否发送成功
*/
public boolean sendToAppDataChannel(byte[] data)
```

## 渲染组件 RtcRender

当创建 RtcClient 时需要传入渲染组件参数，RtcRender 封装了渲染相关操作。将该渲染组件传入 RtcClient 的构造函数即可，在 RtcClient 中将完成初始化等操作。

```xml
<com.pxy.lib_sr.render.RtcRender
    android:id="@+id/signal_render"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_gravity="center_vertical|center_horizontal"
    android:layout_margin="0dp"
    android:background="@android:color/transparent"
    android:padding="0dp" />
```

### 监听视频帧，获取一帧图像

通过 addFrameListener 添加监听器，当收到视频图像后，每给监听器将回调一帧图像。

```java
public void addFrameListener(
            RtcEglRenderer.FrameListener listener, float scale, RendererCommon.GlDrawer drawerParam) 
public void addFrameListener(RtcEglRenderer.FrameListener listener, float scale) 
```

例如：

```java 
/**
    * 监听并获取一帧视频
    */
public void onTestCaptureVideoFrame(View view) {
    mRender.addFrameListener(new RtcEglRenderer.FrameListener() {
        @Override
        public void onFrame(Bitmap frame) {
            Log.d(TAG, "on frame width " + frame.getWidth() + " height " + frame.getHeight());
            runOnUiThread(()->{
                mTestCaptureImageView.setImageBitmap(frame);
            });
        }
    }, 1.0f);
}
```

### 监听获取原始图像

通过 addRawVideoFrameListener 添加原始图像监听器。当收到视频帧之后每一帧都将回调给侦听器。

```java
public void addRawVideoFrameListener(VideoSink videoSink)
```

例如：

```java
mRender.addRawVideoFrameListener(new VideoSink() {
    @Override
    public void onFrame(VideoFrame videoFrame) {
        if (videoFrame.getBuffer() instanceof VideoFrame.I420Buffer) {
            // 收到 i420 类型的buffer
            VideoFrame.I420Buffer i420Buffer = (VideoFrame.I420Buffer) videoFrame.getBuffer();
            Log.d(TAG, "on i420 buffer ");
        }  else if (videoFrame.getBuffer() instanceof VideoFrame.TextureBuffer) {
            // 收到opengl原生纹理
            VideoFrame.TextureBuffer textureBuffer = (VideoFrame.TextureBuffer) videoFrame.getBuffer();
            Log.d(TAG, "on texture buffer id " + textureBuffer.getTextureId() + " " + textureBuffer.getType());
        }
        Log.d(TAG, "on frame " + videoFrame.getBuffer().getWidth() + " " + videoFrame.getBuffer().getHeight());
    }
});
```

### 触摸屏相关操作

1. 从后台接口获取应用是否支持的参数

在云雀后台设置某应用支持触摸屏操作

![](doc/touchscreen.png)

2. 通过 `EnterAppliInfo` 相关方法自动解析应用相关参数中新增 `RtcClient.Config.touchOperateMode` 参数，当该参数值为 `mouse` 时表示该应用不支持触摸屏的协议，只能将触摸操作转换成鼠标操作，当该值为 `touchScreen` 时表示该应用支持触摸操作。在进入应用后可通过参数进行相关处理, 例如:

```java
// 判断云端应用是否支持触摸屏的方式
mTouchScreenOperateMode = rtcParams.touchOperateMode.equals(RtcClient.TOUCH_OPERATE_MODE_TOUCHSCREEN);
```

3. `RtcClient` 累新增触摸屏相关接口:

```java
/**
    * 向云端发送触摸按下事件。云端应用支持触摸屏时才会有相应。
    * @param px 按下的手指的 x
    * @param py 按下的手指的 y
    * @param timestamp 时间戳，毫秒
    * @param id 当前触摸事件按下手指的 id，同一个手指 id 相同。id 应每次触摸增加。即 touchdown 时 +1，touch
    *           move 和 touchup 时保持。下一次又手指 touchdown 时再 +1 获得新的 id
    * @return 是否发送成功
    */
public boolean sendTouchDown(int px, int py, long timestamp, int id)

/**
    * 向云端发送触摸按下事件。云端应用支持触摸屏时才会有相应。
    * @param touchDown {@see sendTouchDown(int px, int py, long timestamp, int id)}
    * @return 是否发送成功
*/
public boolean sendTouchDown(ClientInput.TouchDown touchDown)

/**
    * 向云端发送触摸移动事件。云端应用支持触摸屏时才会有相应。
    * @param px 按下的手指的 x
    * @param py 按下的手指的 y
    * @param timestamp 时间戳，毫秒
    * @param id 当前触摸事件按下手指的 id，同一个手指 id 相同。id 应每次触摸增加。即 touchdown 时 +1，touch
    *           move 和 touchup 时保持。下一次又手指 touchdown 时再 +1 获得新的 id
    * @return 是否发送成功
    */
public boolean sendTouchMove(int px, int py, long timestamp, int id)

/**
    * 向云端发送触摸移动事件。云端应用支持触摸屏时才会有相应。
    * @param touchMove {@see sendTouchDown(int px, int py, long timestamp, int id)}
    * @return 是否发送成功
    */
public boolean sendTouchMove(ClientInput.TouchMove touchMove) 

/**
    * 向云端发送触摸抬起事件。云端应用支持触摸屏时才会有相应。
    * @param px 按下的手指的 x
    * @param py 按下的手指的 y
    * @param timestamp 时间戳，毫秒
    * @param id 当前触摸事件按下手指的 id，同一个手指 id 相同。id 应每次触摸增加。即 touchdown 时 +1，touch
    *           move 和 touchup 时保持。下一次又手指 touchdown 时再 +1 获得新的 id
    * @return 是否发送成功
    */
public boolean sendTouchUp(int px, int py, long timestamp, int id)

/**
    * 向云端发送触摸移动事件。云端应用支持触摸屏时才会有相应。
    * @param touchUp {@see sendTouchUp(int px, int py, long timestamp, int id)}
    * @return 是否发送成功
    */
public boolean sendTouchUp(ClientInput.TouchUp touchUp)
```

4. Android 触摸事件如何转换并发送请参照 Demo 中提供的 TouchScreenHandler 类

