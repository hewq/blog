# 初探 lottie

## 秋有话说

作为一名 H5 前端程序员，在一些包含动画的项目中，设计师总是要来问一下你"这个动画是要导出序列帧还是 GIF"，"导出 GIF 可能会有白边"，"不能再压缩，再压就糊了“…...

**使用序列帧的方式做动画**

在之前的一些项目中，动画一般都是让设计师导出序列帧，然后在前端通过 css 来实现动画，但是你知道的，序列帧文件比较大，而且我是自己把这些序列帧在 ps 上做成 sprite 的，无疑又增加了自己的工作量，如果使用 TexturePacker 自动生成 sprite，但是序列帧是没有按顺序排放的，TexturePacker 为了充分利用图片空间，有些帧还会旋转,这样就无法使用 css 的 `steps` 直接做序列帧动画。再者，对于一些有用户的交互的序列帧，操作起来更加麻烦，序列帧在某些机型上还会出现跳帧的情况。

**使用 GIF 做动画**

使用 GIF 做动画，有着相同的一些问题，文件大，不可控，GIF 会一致循环播放，无法做到单次播放，更不用说用户交互了，有些 GIF 还会有白边，显示效果差。

**使用 svg 做动画**

在前端自己用 svg 做动画的方式只试过一次，设计师先在 AE 上导出 svg 的路径坐标，svg 的坐标是根据设计画的顺序生成的，这就导致在一些路径动画中，维持成本相当大，而且前端直接使用 svg 做的一些动画本身实现的成本也比较高。

总的来说，以上的三种方法，都有这着各自的或多或少的一些缺点。

## 初探 lottie

最近在 GitHub 上发现了 airbnb 一个开源项目 [lottie-web](https://github.com/airbnb/lottie-web)，试了一下，这个库可以很方便地对动画进行操作，直接在 AE 上导出 json 文件操作。但是还是发现了一个问题，对于一些把一些图片导入 AE 做成的动画，在 AE 上导出 json 文件时同时还有导出一个 images 文件夹，里面存放了一些动画中的图片，这个文件夹的大小也是有点大的。然而，这个库还是值得去试一下的。下面对 lottie 的使用方法做一些记录。

### AE 安装 bodymovin 插件

1. [下载安装 ZXP installer](https://aescripts.com/learn/zxp-installer/)
2. [下载 bodymovin 插件](https://github.com/airbnb/lottie-web/blob/master/build/extension/bodymovin.zxp)
3. 把下载好的 bodymovin.zxp 拖到 ZXP installer 中
4. 在 AE 的`首选项->常规`中勾选`允许脚本写入文件和访问网络`
5. 在 AE 中打开`菜单窗口->扩展->Bodymovin`，选择要输出的动画和存放路径，点击`render`渲染输出。
6. 渲染完毕就会看到输出的 json 文件和 images 文件夹。

### 获取 bodymovin 库

你可以在 html 页面上直接调用静态地址`https://cdnjs.com/libraries/bodymovin`，你也可以下载 bodymovin 的 js 源文件，在 AE 打开 bodymovin 插件时有个`Get Player`按钮，可以点击跳转下载源文件。

```html
<script src="js/bodymovin.js" type="text/javascript"></script>
<!-- 或者 -->
<script src="https://cdnjs.com/libraries/bodymovin" type="text/javascript"></script>
```

Bodymovin 在 npm 和 bower 上也可以使用。

```shell
# NPM
npm install lottie-web

# Bower
bower install lottie-web
```

另外，文件`lottie.light.js`只支持播放以 svg 格式导出的动画。

### 使用

调用 `lottie.loadAnimation()`方法启动动画。

```javascript
var animation = bodymovin.loadAnimation({
  container: document.getElementById('lottie'), // Required
  path: 'data.json', // Required
  renderer: 'svg/canvas/html', // Required
  loop: true, // Optional
  autoplay: true, // Optional
  name: "Hello World", // Name for future reference. Optional.
})
```

- container：动画容器，dom 元素
- path：从 AE 中导出的 json 文件
- renderer：动画格式
- loop：是否循环播放动画
- autoplay： 是否自动播放动画
- name：动画名称

**动画实例的主要方法**

`anim.play()` — 播放动画。

`anim.stop() `— 停止动画。

`anim.pause() `— 暂停动画。

`anim.setSpeed(speed)` — 设置播放速度，参数 speed 为 Number ，`1`为正常速度。

```javascript
$('.speed-slow').click(function () {
	anim.setSpeed(0.5);
});

$('.speed-normal').click(function () {
	anim.setSpeed(1);
});

$('.speed-fast').click(function () {
	anim.setSpeed(2);
});
```

`anim.goToAndStop(value, isFrame)` — 跳到某一帧并暂停播放。第一个参数是 Number 。第二个参数是 Boolean，设置`true`则表明第一个参数代表的是帧数，`false`代表第一个参数为时间值（单位毫秒），默认 false。

```javascript
$('.gostop').click(function () {
	anim.goToAndStop(3000, false);
});
```

`anim.goToAndPlay(value, isFrame)` — 跳到某一帧并播放。同上。

```javascript
$('.goplay').click(function () {
	anim.goToAndPlay(1000, true);
});
```

`anim.setDirection(direction)` — 设置播放方向。`1` 为正着播，`-1`反着播。

```javascript
$('.dirc').click(function () {
	anim.setDirection(-1);
});
```

`anim.playSegments(segments, forceFlag)` — 播放某一片段。第一个参数为一维数组或多维数组，每个数组包含两个值（开始帧，结束帧），第二个参数是一个 Boolean ，决定是否立即强制播放该片段。

```javascript
$('.seg1').click(function () {
	anim.playSegments([10, 20], false);
});

$('.seg2').click(function () {
	anim.playSegments([[0, 10], [70, 80]], true);
});
```

`anima.destroy()` — 注销动画。

**lottie 的主要方法**

`lottie.play(<name>)` — 播放所有动画，`name`为动画名称，播放该动画。

`lottie.stop(<name>)` — 停止所以动画，`name`为动画名称，停止该动画。

`lottie.setSpeed(speed, <name>)` — 设置动画播放速度，`speed`指定速度，`1`为正常值，`name`为动画名称，只设定该动画的速度。

```javascript
$('.lottie-slow').click(function () {
	lottie.setSpeed(0.5);
});

$('.lottie-normal').click(function () {
	lottie.setSpeed(1);
});

$('.lottie-fast').click(function () {
	lottie.setSpeed(2);
});
```

`lottie.setDirection(direction, <name>)` — 设置动画播放方向，`direction`指定方向，`1`为正向，`-1`为反向，`name`为动画名称，只设定该动画。

`lottie.loadAnimation(obj)` — 加载动画

`lottie.destroy()` — 注销动画释放资源，同时清空 dom 节点。

`lottie.registerAnimation()` — 可以给一个节点直接注册一个动画，该节点必须包含有指向 json 文件的`data-animation-path`属性。

`lottie.setQuality()` — 设置播放质量，默认 high，可以设置为 high、medium、low 或者一个大于 1 的数值。在有些动画中，数值低于 2 时不会有区别。

**事件监听**

`complete` — 动画播放结束时触发（循环播放不会触发）

```javascript
anim.addEventListener('complete', function () {
	console.log('complete');
});
```

`loopComplete` — 进入下一个循环时触发

```javascript
anim.addEventListener('loopComplete', function () {
	console.log('loop complete');
});

```

`enterFrame` — 每进入一帧触发一次

```javascript
anim.addEventListener('enterFrame', function () {
	console.log('enter frame');
});
```

`segmentStart` — 进入片段播放时触发

```javascript
anim.addEventListener('segmentStart', function () {
	console.log('segment start');
});
```

`config_ready` — 初始化配置完成时触发

```javascript
anim.addEventListener('config_ready', function () {
	console.log('config ready');
});
```

`data_ready` — 动画所有部分加载完成时触发

```javascript
anim.addEventListener('data_ready', function () {
	console.log('data ready');
});
```

`DOMLoaded` — dom 元素加载完成时触发

```javascript
anim.addEventListener('DOMLoaded', function () {
	console.log('dom loaded');
});
```

`destroy` — 注销动画时触发

```javascript
anim.addEventListener('destroy', function () {
	console.log('destroy');
});
```

[code](https://github.com/hewq/Front-end/blob/master/apps/a2019/lottie/index.html)