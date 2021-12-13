# 云雀安卓客户端SDK更新说明

## V3.1.0.0

1.接入3.1.0.0协议。

## V3.1.0.1

1.修复连接新版本 websocket 代理失败的问题。

## V3.1.2.0

### 更新点汇总

1. 适配低版本 Android,目前最低适配 Android SDK 版本为 19。

2. 修复在低版本 Android 设备上异常和怪异行为等问题。

3. 修复在华为盒子上运行异常和解码失败的问题（此项修复必须更新lib文件夹下的 webrtc 库，注意不能使用官方发布的 webrtc 库版本）。

4. 添加多人互动模式和相关接口。

5. 添加背景颜色的参数解析，添加loading背景图设置，修改loading时logo载入。

6. 修改鼠标键盘操作相关接口。添加 Windows 键盘按键消息以及和 Android 键盘消息按键的映射。 修复相关问题。（当前版本服务端支持的操作协议跟以前相同还是鼠标键盘)

7. Demo 中添加电视盒子上的手柄，遥控器等设备映射为鼠标键盘消息的演示模式。（当前 Demo 主要兼容电视盒子操作，在手机等触摸设备上操作下一版本添加）。

8. 移动原有 SDK 中关于手势和模拟摇杆的相关控件到 Demo 中。

9. 修复编译打包发布等其他问题。

### 依赖库更新

1. webrtc 编译更新。见 libs 文件夹下 libwebrtc_20200816.arr. 必须更新。

2. okhttp 更新为 3.12+，为适配低版本 Android 设备。低版本目标设备必须更新。

3. Demo 种添加 glide 依赖，用于解析显示封面图，可选更新。

4. 移除 protobuf-java 依赖。

### 一般接口更新

1. 设置 loading 背景图

```java
// 3.1.2.0 新增，设置载入时背景图,设置为 null 时清空背景图。
// 注意设置是背景图不是中间的logo。logo根据后台设置自动载入。
CloudlarkManager.setLoadingBgBigmap(BitmapFactory.decodeResource(getResources(), R.mipmap.bg));
```

2. ScheduleTaskManager 设置请求时间间隔

```java
// 3.1.2.0 添加参数，时间任务的时间间隔。
mScheduleTaskManager = new ScheduleTaskManager(ScheduleTaskManager.SCHEDULE_TIME_SECOND_MS * 2);
```

调用应用列表接口时，新增 getAppliListSync 接口，同步获取列表。

```java
mScheduleTaskManager.addTask(()-> mGetAppliList.getAppliListSync());
```

3. 解析空白部分背景颜色. 进入云端应用后 RtcClient.Config 添加后台空背景颜色设置的解析。解析成功后可设置为 activity 的背景颜色。

```java
int color = Color.parseColor(rtcParams.bgColor);
```

4. WindowsKeyCodes. 添加 Android 键盘到 Windows 键盘虚拟码的转换。和 Windows 虚拟键盘码的值。

```java
int vkey = WindowsKeyCodes.KEYCODE_W;
mRtcClient.sendKeyDown(vkey, false);
mRtcClient.sendKeyUp(vkey);
```

### 多人互动模式

1. 使用互动模式进入应用。主要通过 EnterAppliInfo 类配置相关参数。

在多人互动模式下用户有两种基本角色，房间的创建者即第一个创建并进入应用的用户和观看者即后来通过分享码进入的用户。

> 您的云雀系统是否支持相关模式请咨询商务人员。

普通模式下与之前相同

```java
EnterAppliInfo enterAppliInfo = new EnterAppliInfo(mCallback);
enterAppliInfo.setFrameRate(mFrameRate);
enterAppliInfo.setCodeRate(mCodeRate);
```

启用互动模式，设置类型.

```java
// PlayerMode -》 PLAYER_MODE_NORMAL 普通模式，默认值，没有观看者
//                PLAYER_MODE_INTERACTIVE 互动模式，只有一个操作权。
//                PLAYER_MODE_INTERACTIVE_MUTIPLAYER 多人互动模式，所有玩家都有操作权
enterAppliInfo.setPlayerMode(RtcClient.PLAYER_MODE_INTERACTIVE);
```

设置昵称

```java
// 互动模式时需要
enterAppliInfo.setNickName(nickName);
```

#### 房间创建者的设置

设置用户类型。互动模式和多人互动模式时必须设置。当用户为演示者即当前房间的创建者时应传 USER_TYPE_PLAYER.

```java
// 互动模式时设置用户是观看者还是操作者
// USER_TYPE_PLAYER 操作者
// USER_TYPE_OBSERVER 观看者
enterAppliInfo.setUserType(RtcClient.USER_TYPE_PLAYER);
```

房间创建者设置分享方式。当未设置房间码时将自动通过 taskId 分享。即 `setRoomCode` 为传入空字符传。

当创建者调用创建房间码接口时，`setRoomCode` 应传入房间码。

```java
// 生成口令
private void genRoomCode(final View view) {
    RoomCode roomCode = new RoomCode();
    roomCode.getRoomCode(new RoomCode.Callback() {
        @Override
        public void onSuccess(String roomCode) {
            mRoomCode = roomCode;
            mContext.runOnUiThread(()->{
                mRoomCodeTextView.setVisibility(View.VISIBLE);
                mRoomCodeTextView.setText(mRoomCode);
            });
        }

        @Override
        public void onFail(String err) {
            toastInner("口令生成失败 " + err);
        }
    });
}
```

```java
// 创建者设置口令
enterAppliInfo.setRoomCode(mRoomCode);
```

#### 加入房间的玩家的设置

观看者 USER_TYPE_OBSERVER。

```java
enterAppliInfo.setUserType(RtcClient.USER_TYPE_OBSERVER);
```

观看者选择使用何种方式进入房间

```java
if (mObTaskIdMode) {
    enterAppliInfo.setRoomCode(obRoomCode);
} else {
    enterAppliInfo.setTaskId(obRoomCode);
}
```

#### 进入应用

参数设置好后同普通模式一样进入应用

```java
enterAppliInfo.enterApp(mItem);
```

当进入应用成功之后，通过 RtcClient 类的相关接口和事件分配玩家操作权等。

RtcClient.RtcClientEvent 新增回调事件，onLoginSuccess，返回当前用户的 id。

```java
@Override
public void onLoginSuccess(int uid) {
    Log.d(TAG, "onLoginSuccess " + uid);
    mUserId = uid;
}
```

RtcClient.RtcClientEvent 新增 onPlayerList 回调事件，获得玩家列表。

```java
/**
* 3.1.2.0 新增
* @param playerList 多人互动模式时用户列表
*/
@Override
public void onPlayerList(List<AppNotification.PlayerDesc> playerList) {
    if (mPlayerListAdapter == null) {
        Log.w(TAG, "no playerlist adapter found");
        return;
    }
    Log.d(TAG, "onPlayerList " + playerList);
    runOnUiThread(() -> {
        if (mPlayerListAdapter.getCount() < playerList.size()) {
            toastInner("有观看者加入");
        }
        mPlayerListAdapter.fresh(playerList);
    });
}
```

其中 PlayerDesc 主要包含一下属性

```java
// 玩家的 id
public int getId() {
    return this.id;
}
// 玩家的昵称
public String getNickName() {
    return this.nickName;
}
// 是否时当前房间的拥有者
public boolean isTaskOwner() {
    return this.taskOwner;
}
// 是否有控制权
public boolean isController() {
    return this.controller;
}
```

调用 RtcClient 实例的 dispatchController 方法赋予玩家控制权。需要注意只有房间拥有着才有权限切换控制权限。

```java
boolean res = mRtcClient.dispatchController(playerDesc.getId());
```

> 多人互动模式配置参考具体使用 InterActiveDialog 类。

### RtcClient 接口更改

1. 回调函数更新

```java
/**
* 登录服务器成功
*/
void onLoginSuccess(int uid);
/**
* 玩家列表
*/
void onPlayerList(List<AppNotification.PlayerDesc> playerList);
/**
* 视频连接状态报告.3.1.2.0 修复没有数据的问题。
*/
void onPeerStatusReport(SampleRTCStats sampleRTCStats);
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

### Demo 更新

1.当释放云端应用 activity 时，要手动释放 RtcRender。调用实例的 release 方法，避免 gl 资源泄露。参考 demo 中 RtcActivity onDestroy.

```java
// 释放 gl 资源。
mRender.release();
```

2. 输入设备相关

Demo 中添加了相关输入设备的映射，内容参考 app\src\main\res\xml 下的 xml配置。

解析方法参考 app\src\main\java\com\pxy\larksr_demo\inputs 下相关类。

## V3.1.2.2

1. 添加 https 支持。

设置服务器地址时可以传入参数是否启用 https。
也可以直接传入带协议头的 url 如 `https:\\test.test.com`。如果支持传入带协议头的 url，将覆盖 useHttps 参数。
如果传入 `test.test.com`, 则将使用 useHttps 参数确认是否使用 https.

```java
Base.setServerAddr(useHttps, serverIp);
```

2.修复某些机型序列化消息失败的问题。

3.修复其他问题。

## V3.1.2.3

1. 对应服务器新版心跳包机制。

2. 升级 webrtc 库。

3. 修复其他问题。

## V3.1.3.0

更新列表：
1.添加SDK授权校验
2.添加手柄相关接口
3.GetAppliList 添加翻页相关参数

### SDK授权校验

新增授权校验接口，当 SDK 初始化之后应调用授权校验接口验证授权 ID。初始化授权 ID 失败将抛出异常。
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
* @param key
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

## V3.1.3.1

更新列表：
1.RtcClient.Config 添加 useGamepad 参数
2.EnterAppliInfo 添加 useGamepad 解析
3.AppListItem 添加 useGamepad 参数

当后台指定某应用支持手柄操作时，可通过 AppListItem/EnterAppliInfo 获取 useGamepad 判断是否支持。

## V3.1.3.3

更新列表:
1. 添加数据通道相关回调和发送接口
2. 添加输入文字相关回调和发送接口
3. 添加 RtcRender 监听原始视频帧
4. 添加视频大小改变和准备好回调

## V3.1.7.0

更新列表:
1. 新增一个初始化接口 CloudlarkManager.init(Context context, String type, boolean isTV),表明是否是电视盒子版本
2. 新增触摸屏相关操作接口和解析参数

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
