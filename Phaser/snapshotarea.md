# Phaser 画布截图

现在很多在微信做推广的 h5 经常需要用到长按保存活动图片的功能，Phaser 就有这样一个 api 可以直接用来截图保存

```html
<!DOCTYPE html>
<html>
<head>
  <!-- 引入 phaser -->
  <script src="https://cdn.jsdelivr.net/npm/phaser@3.15.1/dist/phaser-arcade-physics.min.js"></script>
</head>
  <body>
    <div id="game" class="game"></div>
    <div class="snapshot"></div>
    <script>
      let PlayGame = new Phaser.Class({
        Extends: Phaser.Scene,
        initialize: function PlayGame() {
            Phaser.Scene.call(this, {
                key: 'playGame',
                active: false
            })
        },
        preload() {
          this.load.image('bg', 'bg.jpg')
        },
        snapshot() {
            return new Promise((resolve) => {
                // 主要就是这句代码
                this.game.renderer.snapshotArea(0, 0, 750, 1464, (image) => {
                    $('.snapshot').append(image)
                    resolve()
                }, 'image/jpeg')
            })
        },
        create() {
          this.add.image(game.config.width / 2, game.config.height / 2, 'bg')
          this.snapshot()
        }
      })

      let config = {
        type: Phaser.AUTO,
        parent: 'game',
        width: 750,
        height: 1464,
        transparent: false,
        backgroundColor: '#000000',
        scene: PlayGame
      }
      let game = new Phaser.Game(config)
    </script>
  </body>
</html>
```

上面的核心代码是 `snapshotArea` 方法，使用这个方法就可以实现对画布的截图。

- `snapshotArea(x, y, width, height, callback[, type][, encoderOptions])`
  - `x, y` ：截图的起始坐标
  - `width, height` ：截图的宽高
  - `callback` ：截图后的回调方法
  - `type` ：图像格式，有 `image/png` 和 `image/jpeg` 可选，默认 `image/png`
  - `encoderOptions` ：图片质量，用于有损压缩的图像格式，介于 0 和 1 之间

另外要注意的是，游戏每一帧只能激活一个快照，所以如果要截图多张图片的时候一定要在上一次截图完成后再调用截图的相关方法。上面的代码用 `Promise` 把 `snapshotArea` 包裹起来，避免回调地狱

***tips: 如果使用 `image/png` 格式保存的图片有白边的话，可以使用 `image/jpeg` 格式，避免出现白边***

[更多截图相关的 API， 可自行查阅](https://photonstorm.github.io/phaser3-docs/Phaser.Renderer.Canvas.CanvasRenderer.html#snapshotCanvas__anchor)
