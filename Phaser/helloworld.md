# Phaser 的基本使用

```html
<!DOCTYPE html>
<html>
<head>
  <!-- 引入 phaser -->
  <script src="https://cdn.jsdelivr.net/npm/phaser@3.15.1/dist/phaser-arcade-physics.min.js"></script>
</head>
  <body>
    <div id="game" class="game"></div>
    <script>
      let config = {
        type: Phaser.AUTO,
        parent: 'game',
        width: 750,
        height: 1464,
        transparent: false,
        backgroundColor: '#000000',
        scene: {
          preload: preload,
          create: create
        }
      }
      let game = new Phaser.Game(config)

      let preload = function () {
        this.load.image('key', 'url')
      }

      let create = function () {
        this.add.image(100, 100, 'key')
      }
    </script>
  </body>
</html>
```

以上就是一个简单的 Phaser 小demo

- `Phaser.Game` 实例是整个 Phaser 游戏的主控制器。它负责处理启动过程，解析配置值，创建渲染器以及设置所有全局相位器系统，例如声音和输入。完成后，它将启动场景管理器，然后开始主游戏。**简单来说，就是游戏的总开关，用来启动游戏**

- `config` 是[游戏的配置](https://photonstorm.github.io/phaser3-docs/Phaser.Types.Core.html#.GameConfig)
  - `type` : number，渲染器类型（Phaser.AUTO, Phaser.CANVAS, Phaser.HEADLESS, Phaser.WEBGL），Phaser.AUTO 会优先选择 webgl，如果不支持 webgl 则选择 canvas
  - `parent` : HTMLElement | string，游戏画布的父节点，如果不指定则直接插入`body`中
  - `width` ：integer | string，游戏画布的宽度，以像素为单位
  - `height` ： integer | string，游戏画布的高度，以像素为单位
  - `transparent` ：boolean，设置游戏画布背景是否透明
  - `backgroundColor` ： string | number，游戏画布的背景颜色
  - `scene` ：[Phaser.Types.Scenes.CreateSceneFromObjectConfig](https://photonstorm.github.io/phaser3-docs/Phaser.Types.Scenes.html#.CreateSceneFromObjectConfig) | Phaser.Scene | function | Array，添加到游戏中的场景，如果有多个则显示第一个
    - `preload` ：场景的预加载回调函数
    - `create`： 场景创建的回调函数，在 `init` 和 `preload` 之后调用此方法
