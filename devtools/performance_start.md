# Chrome 开发者工具——Performance 快速入门

> 原文链接：<https://developers.google.com/web/tools/chrome-devtools/evaluate-performance?hl=zh-cn>
> 如果你想学会如何使用 Chrome 的 Performance 来分析页面性能，而你有无法使用 Google，也许这篇文章能帮到你。
> 这篇文章将教会你如何使用 Performance 面板来分析浏览器运行时的性能。

<!-- TOC -->

- [Chrome 开发者工具——Performance 快速入门](#chrome-开发者工具performance-快速入门)
  - [开始](#开始)
  - [模拟手机 CPU](#模拟手机-cpu)
  - [设置 Demo](#设置-demo)
  - [记录运行时的性能](#记录运行时的性能)
  - [结果分析](#结果分析)
    - [分析 frames per second（FPS）](#分析-frames-per-secondfps)
    - [找到性能瓶颈](#找到性能瓶颈)

<!-- /TOC -->

## 开始

在本教程中，你将在 Chrome 浏览器的开发者工具中，使用 Performance 面板查找页面性能瓶颈。

1. 打开 Chrome 的无痕模式。无痕模式能够确保浏览器在没有其他干扰的环境中运行。比如，如果你的 Chrome 安装了很多插件，这些插件可能会干扰到性能的评估。
2. 在无痕窗口中打开 <https://hewq.github.io/apps/a20201027chromedevtool/index.html>，这是一个用来分析性能的 Demo。
3. 按下`Command+Option+I`(Mac)或者`Control+Shift+I`(Windows, Linux)打开开发者工具。
![chrome devtool](get-started.png)

## 模拟手机 CPU

手机的 CPU 性能比起电脑的要差一些，所以当你分析页面的时候可以通过限制 CPU 来模拟页面在手机上的表现。

1. 在开发者工具中，点击 **Performance**。
2. 勾选 **Screenshots** 复选框。
3. 点击右边的 Capture Settings![Capture Settings](capture-settings.png)，将会看到有关性能指标的设置。
4. 在 CPU 那一栏，选择 **2x slowdown** （新版本的 Chrome 可能只有 4x slowdown 和 6x slowdown）。限制 CPU 比正常情况下慢 2 倍。
![throttling](throttling.svg)

## 设置 Demo

很难做到打开该页面的所有用户都能有一样的性能演示，所以你可以手动设置，无论使用哪种设置，确保你的体验和本教程中看到的屏幕截图和说明相近。

1. 持续点击 **Add 10**，直到蓝色方块的移动明显变慢。
2. 点击 **Optimize**，蓝色方块将移动得更快更顺滑。
3. 点击 **Un-Optimize**，蓝色方块的移动又会变慢变卡。

## 记录运行时的性能

当你开启优化（Optimize）的时候，蓝色的方块比没有开启优化（Un-Optimize）时移动更快。为什么会这样？两个版本都是在相同的时间内将方块移动到相同位置。在 **Performance** 中进行记录，以了解如何检测未优化版本的性能瓶颈。

1. 点击 **Record** ![record](record.png)。开发者工具将会记录页面运行时的性能指标。![profiling](profiling.png)
2. 等待几秒钟。
3. 点击 **Stop**。开发者工具停止记录，处理数据，然后显示结果。![result](results.png)

是不是看到了很多眼花缭乱的数据无从下手，不用担心，接下来你就能学会如何分析这些数据。

## 结果分析

记录了页面性能之后，就可以衡量页面的性能有多差，并找出原因。

### 分析 frames per second（FPS）

衡量动画的主要指标就是 frames per second（FPS）。当动画以 60 FPS 运行的时候，用户就会觉得很顺滑。

1. 查看 **FPS** 图表。当你看到 **FPS** 上面的红色条时，就表示帧速率下降得很低，可能会影响用户的体验。一般来说，绿色条越高，FPS 越高。![fps](fps-chart.svg)
2. 在 **FPS** 图表下面有个 **CPU** 图表。**CPU** 图表中的颜色和下方 **Summary** 的颜色时相对应的。**CPU** 图表充满颜色意味着在记录过程中，CPU 的使用已经达到最大值。当你看到 CPU 长时间处于满负荷状态时，你应该想办法降低 CPU 的使用。![cpu](cpu-summary.svg)
3. 鼠标放到 **FPS**、**CPU** 或者 **NET** 图表上，开发者工具将显示出该时间点的页面截图。将鼠标从左到右移动，就可以重新看到记录过程中页面的渲染情况。这对于手动分析动画的进度很有用。![screenshot](screenshot.png)
4. 在 **Frames** 部分，将鼠标放到任意一个绿色方块上面，你就能看到指定帧的 FPS。每一帧都可能远低于 60 FPS。![frame](frame.png)

当然，在这个 Demo 中，页面性能不好是很明显的。但是在实际情况下，可能并不是那么容易分辨，所以有这些工具进行测量就非常方便了。

***打开 FPS 显示器***

另一个方便的工具就是 FPS 显示器，它可以在页面运行时显示 FPS 的实时数据。

1. 按下`Command+Shift+P`（Mac）或者`Control+Shift+P`（Windows, Linux）打开命令菜单。
2. 在命令菜单中输入`Rendering`，选择 **Show Rendering**。
3. 在 **Rendering** 选项下，选中 **FPS Meter**（或者 Frame Rendering Stats）, FPS 的实时数据就会显示出来。![fps-meter](fps-meter.png)

### 找到性能瓶颈

现在，你已经测量并验证了动画效果不佳，接下来要回答的问题就是：为什么？

1. 留意 Summary 选项，如果没有选择任何事件，这里就会显示活动的细分。这个页面在渲染上花费了大部分的时间，因为性能是一门减少工作量的艺术，所以你的目标是减少页面渲染所花费的时间。![summary](summary.svg)
2. 展开 **Main** 部分。你将看到主线程随时间变化的火焰图。x 轴表示随时间变化的记录，每一条代表一个事件，宽度越长表示花费的时间越多。y 轴表示的是调用栈。当你看到事件相互叠加时，代表上面的事件触发了下面的事件。![main](main.svg)
3. 通过单击，在**概述**上（包含 **FPS**、**CPU** 和 **NET** 图表的那部分）按住并拖动鼠标来放大单个 **Animation Frame Fired** 事件。**Main** 和 **Summary** 部分会只显示所选部分的信息。![zoom](zoomed.png)
***缩放的另一种方法是通过单击主要部分的背景或选择一个事件来聚焦主要部分，然后按W，A，S和D键。**
4. 留意 **Animation Frame Fired** 右上角的三角形，这是一个警告，每当你看到这个三角形时，说明可能存在与此事件相关的问题。
***每当执行requestAnimationFrame（）回调时，都会触发 Animation Frame Fired 事件。**
5. 点击 **Animation Frame Fired** 事件，**Summary** 里会显示与该事件相关的信息。留意 **reveal** 链接，点击该链接，可以看到触发 **Animation Frame Fired** 的事件。同时留意 **app.js:94**（app.js:81） 这个链接，点击会跳到源代码中的相关行。![animation](animation-frame-fired.png)
***选择事件后，使用箭头键选择它旁边的事件。**
6. 在 **app.update** 下面，有一堆紫色的事件，如果它们足够宽，看上去好像每一个上面都有一个红色三角形。点击紫色的 **Layout** 事件，**Summary** 中显示了更多关于该事件的信息。实际上，是一个关于强制回流（布局）的警告。
7. 在 **Summary** 中点击 **Layout Forced** 下的 **app.js:70**（app.js:57） 定位到强制回流的代码上。![forced](forced-layout-src.png)
***这个代码的问题是，在每个动画帧中，它会更改每个方块的样式，然后查询页面上每个方块的位置。因为样式发生了变化，浏览器不知道每个正方形的位置是否发生了变化，因此必须重新布局正方形以计算其位置。**

需要了解的内容有很多，但是现在你已经有了分析运行时性能的基本流程的基础。加油！

***分析优化版本***
使用刚刚学到的工具和流程，点击页面上 **Optimize** 启用优化后的代码，获取另一个性能记录，然后分析结果。从提高帧率到 **Main** 火焰图中事件的减少，你可以看到，优化后的页面工作量大大减少，从而提高了性能。

***即使这个“优化”版本也不是那么好，因为它仍然操纵每个正方形的 top 属性。更好的方法是使用只影响合成层的属性。**
