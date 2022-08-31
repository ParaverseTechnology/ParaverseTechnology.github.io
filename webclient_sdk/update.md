# 更新

## V3.2.30

> V3.2.30 仅支持服务端 [V3.2.3.1](https://www.pingxingyun.com/devCenter.html) 以上版本。
> [老版本 SDK Demo 下载](https://github.com/pingxingyun/lark_sr_websdk_demos/releases/tag/V3.2.10)

1. [添加像素流送支持](https://docs.unrealengine.com/4.27/zh-CN/SharingAndReleasing/PixelStreaming/PixelStreamingIntro/)。
2. 完善安全性, 添加 SDK ID 加密配置。
3. 移除 CreateLarkSRClientFromePXYHost，CreateLarkSRClientFromeAPI, CreateLarkSRClientFromeUrl 函数。直接使用 `new LarkSR(config: ILarkSRConfig)` 方式创建对象。

## V3.2.33

1. 添加智能语音相关接口支持

LarkSR 对象添加:

```typescript
/**
 * 开始智能语音对话文本输入
 * 应在只能语音为开始状态下开始输入
 * @param text 输入的文本
 * @returns 返回对话 ID
 */
public aiDmTextInput(text: string) {
/**
 * 开始智能语音对话语音输入，自动请求录音权限
 * 应在只能语音为开始状态下开始输入
 * @returns 成功返回对话id，失败返回异常
 */
public startAiDmVoiceInput() {

/**
 * 停止智能语音输入，通知语音引擎当前用户输入对话结束
 * @returns 返回当前对话的id
 */
public stopAiDmVoiceInput() {
```

LarkSR 添加如下事件:

```typescript
larksr.on('aivoicestatus', (e) => {
    // 智能语音服务状态
});

larksr.on('aivoiceasrresult', (e) => {
    // 智能语音转文字识别结果
});

larksr.on('aivoicedmresult', (e) => {
    // 智能语音对话结果
});
```

## V3.2.37

```typescript
/**
 * 设置是否强制横屏显示内容.
 * handelRootElementSize 必须设置为 true 才有作用。
 * 要注意强制横屏模式下网页的坐标系xy和视觉上相反，如果通过外部输入 input 事件。要注意调整
 * @param force 是否强制横屏
 */
setMobileForceLandScape(force: boolean): void;
```

## V3.2.311

新增麦克风输入接口。客户端打开后云端应用可直接通过读取声卡上的麦克风接收到音频。

> 该功能匹配的服务端版本最低为 V3.2.51
> 使用该功能要注意在后台应用管理中开启音频输入功能
> 注意要在连接成功之后打开媒体设备。即 `mediaPlayed` 之后调用才有效

```typescript
/**
 * 打开一个音频设备，要注意浏览器限制在 https 或者 localhost 下才能打开音频
 * @param deviceId 音频设备id，如果不传将打开默认设备。@see getConnectedAudioinputDevices
 * @returns
 */
openAudio(deviceId?: string);
/**
 * 关闭当前的音频设备
 * @returns
 */
closeAudio(): boolean | undefined;
/**
 * 返回已连接的音频设备列表，设备列表中的设备的 deviceId 可用来打开某个音频设备
 * @returns
 */
getConnectedAudioinputDevices(): Promise<MediaDeviceInfo[] | undefined>;
/**
 * 设置当前已开启的音频track是否启用状态
 * @param enable 是否启用
 * @returns
 */
setAudioEnable(enable: boolean): void | undefined;
```

larksr 配置项新增自动打开麦克风配置,在`new LarkSR({ ... 此处省略其他配置 ... audioInputAutoStart: true})` 时传入，

> 手动传入的 `audioInputAutoStart` 优先级高于后台配置

```javascript
/**
* 当启用音频输入功能，是否自动连入音频设备。
* 默认关闭。
* 需要注意默认打开的是系统中默认的音频设备。
*/
audioInputAutoStart?: boolean;
```

## V3.2.319

新增视频输入接口。客户端打开后云端应用可直接通过读取服务端的摄像头读取视频数据。

> 该功能匹配的服务端版本最低为 V3.2.61
> 使用该功能要注意在后台应用管理中开启视频输入功能
> 注意要在连接成功之后打开媒体设备。即 `mediaPlayed` 之后调用才有效

```typescript
/**
 * 打开一个视频设备，要注意浏览器限制在 https 或者 localhost 下才能打开视频
 * @param audio boolean 是否同时开启音频
 * @param cameraId 视频设备id，如果不传将打开默认设备。@see getConnectedVideoinputDevices
 * @returns Promise
 */
openVideo(audio?: boolean, cameraId?: string);
/**
 * 打开默认媒体设备，要注意浏览器限制在 https 或者 localhost 下
 * 如果需要指定特殊的媒体设备请单独使用 @see openAudio @see openVideo
 * @param video 是否包含视频
 * @param audio 是否包含音频
 * @returns Promise
 */
openDefaultMedia(video?: boolean, audio?: boolean)>;
/**
 * 返回已连接的视频设备
 * @returns Promise<MediaDeviceInfo[]>
 */
getConnectedVideoinputDevices();
/**
 * 设置当前已开启的视频track是否启用状态
 * @param enable 是否启用
 * @returns void
 */
setVideoEnable(enable: boolean);
```

larksr 配置项新增自动打开视频输入配置,在`new LarkSR({ ... 此处省略其他配置 ... videoInputAutoStart: true})` 时传入

> 手动传入的 `videoInputAutoStart` 优先级高于后台配置

```javascript
/**
 * 当启用视频输入功能，是否自动连入视频设备。
 * 默认关闭。
 * 需要注意默认打开的是系统中默认的视频设备。
 */
videoInputAutoStart?: boolean;
```

## V3.2.321

larksr 配置项新增默认放大手势和鼠标滚轮对应关系和是否使用触摸屏模式,在`new LarkSR({ ... 此处省略其他配置 ... mouseZoomDirection: 0， touchOperateMode: "mouse"})` 时传入

> 手动传入的配置优先级高于后台配置

```javascript
/**
 * mouseZoomDirection
 * 用于移动端捏合缩放操作与应用鼠标缩放的对应关系
 * 1:鼠标滚轮向上为放大，
 * 0:鼠标滚轮向下为放大(default)
 */
mouseZoomDirection?: number;
/**
 * 触摸指令模式，对应后台应用管理->应用编辑->移动端高级设置->触摸指令模式 优先级高于后台配置
 * 触摸指令，移动端的触摸指令对应为云端的触屏还是鼠标
 * 'touchScreen' | 'mouse'
 */
touchOperateMode?: 'touchScreen' | 'mouse';
```

captrueFrame 方法会将截图和通过返回值带出来

```javascript
/**
 * 采集一帧图像
 * @params data: any 抛出采集事件时抛出的附加data，比如采集的时间戳
 * @return { data: any, base64: base64string } 返回传入的 data 和采集的 base64 字符串
 */
captrueFrame(data: any)
```

新增销毁方法，要注意，销毁之后 larksr 对象不再可用

```javascript
/**
 * 从DOM种删除渲染组件，注意删除渲染组件之后将无法再次进入应用，所有状态将失效,不可恢复，只能重新new LarkSR
 */
destroy(): void;
```

添加操作类中的 keyboardhandler 开放访问，可用配置拦截哪些按键的默认行为

```javascript
// 设置键盘事件默认拦截浏览器默认行为。
// 在 preventKeys 中的将拦截，如果数组设置为空则全部不拦截
// 默认拦截 F1 F5 F12
larksr.op.keyboardHandler.preventKeys = [["F1", "F5", "F12"]];
```

操作类中添加 gestureHandler 公开访问,gestureHandler 处理将移动端触摸事件对应为鼠标事件的过程。

可用通过 gestureHandler 调节触摸事件触发参数

```javascript
// 触摸事件在对应为鼠标事件中，提供的鼠标相对移动的速度。
// 即 rx = （pxNew-pxOld） * relativeMouseMoveSpeed
//    ry = （pyNew-pyOld）* relativeMouseMoveSpeed
// 可能会影响使用相对移动（rawInput）判断的云端应用的效果
larksr.op.gestureHandler.relativeMouseMoveSpeed = 2;
// 触摸模拟为鼠标事件时，模拟点击的判断范围。
// 实际的鼠标事件点击时一般不会移动，而触摸操作基本上一定会触发移动事件
// 如果要模拟PC上的击事件，比如单击或双击，在鼠标按下和抬起之间不应再插入鼠标移动
// 当触发移动事件时，超出该判断范围才发送移动事件。
// 注意，修改该值可能会导致云端应用上判断单击或双击失败，如果云端应用不包含类似操作基本影响不大
larksr.op.gestureHandler.tapLimitRadius = 20;
// 在该时间范围下将尝试发送单机按下事件，修改该值可能会导致单机事件的触发成功率
larksr.op.gestureHandler.tapLimitTimeout = 100;
```

可用通过 gestureHandler 拦截触摸事件回调, 通过 `larksr.op.gestureHandler` 设置回调函数

## V3.2.322

1. 修复某些情况下后台设置的 LOGO 会闪一下默认 LOGO 的问题
2. 添加 `larksr.virtualCursorPosition` 和 `larksr.virtualCursorPositionRaw` (省略 larksr 对象创建代码) 获取当前虚拟鼠标或者触摸点的位置。

## V3.2.324

1. 修复 iOS 屏幕高度判断问题
2. 添加获取应用列表接口 `API.GetAppliList(server: string, params: any);`
3. 添加获取区域列表接口 `API.RegionList(server: string, params: any);`
4. 添加获取后台配置UI接口 `larksr.getTouchCtrMapping()`
5. 添加  `RTMP_STREAM_STATE  = "rtmpstreamstate"` 事件，返回云端直播推流事件
6. 添加 `RTMP_STREAM_ERROR = "rtmpstreamerror"` 事件，返回云端直播推流错误
7. 添加 `larksr.StartCloudLiveStreaming(params)` 接口，启动云端直播推流功能
8. 添加 `larksr.StopLiveStreaming()` 接口，关闭云端直播推流功能.

## V3.2.326

1. 截图接口 `captrueFrame(data: any, option?: { width: number; height: number;})` 添加 option 参数，可以设置截图的宽高
2. 添加 onlyHandleRootElementTransform 参数 `new LarkSR({ ... 此处省略其他配置 ... onlyHandleRootElementTransform: 0})`

```javascript
/**
* 是否只设置根的组件的旋转。只当 handleRootElementSize 为 true 时有效。设置根组件的旋转用于 mobileForceLandscape 模式。
* 默认为false，此时 SDK 会设置根组件的宽高,margin 0,padding 0
* 为true并且handleRootElementSize也为true时，只设置根组件的旋转属性，用于强制横屏模式以及强制横屏竖屏的切换。
* 要注意onlyHandleRootElementTransform开启成功时，要保证根节点的元素大小并且当根节点变化时应调用 resize 方法通知更新根节点的大小。
*/
onlyHandleRootElementTransform?: boolean;
```