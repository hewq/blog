# Phaser 小游戏——小球成长

| 需求 |
| --- |
| 1. 小球在画布内不规则运动，不会发生碰撞，只会重叠在一起。 |
| 2. 有和小球相同数量的按钮。 |
| 3. 每个小球都有自己都编号并且和相应编号的按钮绑定。 |
| 4. 长按按钮，相应的小球会变大。 |
| 5. 松开按钮，小球停止变大。 |
| 6. 如果正在变大的小球和其他任何一个小球相碰，两个小球的大小缩小一半并且停止变大。 |
| 7. 每个球只能变大一次。 |

## 先把简单的框架搭起来

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>growing circle</title>
    <style>
        * {
            margin: 0;
            padding: 0;
        }
    </style>
</head>

<body>
    <div id="game"></div>

    <script src="/public/js/phaser.min.js"></script>
    <script>
        let game
        let gameOptions = {
            ballSpeed: 300, // 小球运动速度
            balls: 8, // 小球总数
            ballRadius: 50, // 小球半径
            growRate: 1 // 增长速率
        }

        class PlayGame extends Phaser.Scene {
            constructor() {
                super('PlayGame')
            }
            preload() {
                this.load.image('ball', 'ball.png')
                this.load.image('button', 'button.png')
            }
            create() {}
            update() {}
        }

        let gameConfig = {
            type: Phaser.AUTO,
            scale: {
                mode: Phaser.Scale.WIDTH_CONTROLS_HEIGHT,
                autoCenter: Phaser.Scale.CENTER_BOTH,
                parent: 'game',
                width: 750,
                height: 1464
            },
            physics: {
                default: 'arcade'
            },
            scene: PlayGame
        }

        game = new Phaser.Game(gameConfig)
    </script>
</body>

</html>
```

引入 `arcade` 物理系统，方便小球做不规则运动

## 放置小球

```js
class PlayGame extends Phaser.Scene {
  create() {
    // 设置小球的运动区域
    this.physics.world.setBounds(0, 0, game.config.width, game.config.width)
    let gameArea = new Phaser.Geom.Rectangle(0, 0, game.config.width, game.config.width)

    // 物理组对象，存放所有的小球
    this.ballGroup = this.physics.add.group()

    // 生成小球
    for (let i = 0; i < gameOptions.balls; i++) {
      // 运动区域内随机一个点
      let randomPosition = Phaser.Geom.Rectangle.Random(gameArea)

      // 放置小球
      let ball = this.ballGroup.create(randomPosition.x, randomPosition.y, 'ball')
      // arcade 物理系统中设置小球的边界为圆形
      ball.setCircle(256)
      ball.setCollideWorldBounds(true)

      // 宽高
      ball.displayHeight = gameOptions.ballRadius
      ball.displayWidth = gameOptions.ballRadius
      // 编号
      ball.index = i
      let ballText = this.add.text(ball.x, ball.y, i, {
        fontFamily: 'Arial',
        fontSize: 24,
        color: '#000000'
      })
      ballText.setOrigin(0.5, 0.5)
    }
  }
}
```

## 放置按钮

```js
class PlayGame extends Phaser.Scene {
  create() {
    let buttonPerRow = gameOptions.balls / 2
    let buttonWidth = game.config.width / buttonPerRow
    this.buttonGroup = this.add.group()

    // 生成按钮
    for (let i < 0; i < gameOptions.balls; i++) {
      let buttonX = buttonWidth * (i % (gameOptions.balls / 2))
      let buttonY = game.config.width + buttonWidth * Math.floor(i / (gameOptions.balls / 2))
      let button = this.add.sprite(buttonX, buttonY, 'button')
      button.setOrigin(0, 0)
      button.displayWidth = buttonWidth
      button.displayHeight = buttonWidth
      button.index = i
      this.buttonGroup.add(button)

      let buttonText = this.add.text(button.getBounds().centerX, button.getBounds().centerY, i, {
        fontFamily: 'Arial',
        fontSize: 64,
        color: '#000000'
      })
      buttonText.setOrigin(0.5, 0.5)
    }
  }
}
```

## 设置分数

```js
class PlayGame extends Phaser.Scene {
  create() {
    this.scoreText = this.add.text(0, game.config.height, 'Score: 0', {
      fontFamily: 'Arial',
      fontSize: 64
    })

    this.scoreText.setOrigin(0, 1)
  }
}
```

## 让小球动起来，不会碰撞，只会重叠

```js
class PlayGame extends Phaser.Scene {
  create() {
    this.ballArray = []
    this.textArray = []

    for (let i < 0; i < gameOptions.balls; i++) {
      this.ballArray.push(ball)
      this.textArray.push(ballText)


      // 随机一个方向向量
      let directionVector = Phaser.Math.RandomXY(new Phaser.Math.Vector2, gameOptions.ballSpeed)
      // 设置小球的运动方向和速度
      ball.setVelocity(directionVector.x, directionVector.y)
      // 设置小球碰到边界反弹
      ball.setBounce(1)
    }
  }
  update() {
    for (let i = 0; i < gameOptions.balls; i++) {
      this.textArray[i].x = this.ballArray[i].x
      this.textArray[i].y = this.ballArray[i].y
    }
  }
}
```

## 给按钮绑定事件

```js
class PlayGame extends Phaser.Scene {
  create() {
    this.input.on('pointerdown', this.startGrowing, this) // 小球开始变大
    this.input.on('pointerup', this.stopGrowing, this) // 小球停止变大
    this.ballToGrow = null
  }
  startGrowing(pointer) {
    this.buttonGroup.getChildren().map(button => {
      if (Phaser.Geom.Rectangle.Contains(button.getBounds(), pointer.x, pointer.y) && button.alpha === 1) {
        button.alpha = 0.5
        this.ballToGrow = button.index
        console.log(button.index)
      }
    })
  }
  stopGrowing() {
    this.ballToGrow = null
  }
  update() {
    this.score = 0
    for (let i = 0; i < gameOptions.balls; i++) {
      this.score += this.ballArray[i].displayWidth - gameOptions.ballRadius;
    }

    this.scoreText.text = 'Score: ' + this.score
    if (this.ballToGrow != null) {
      this.ballArray[this.ballToGrow].displayWidth += gameOptions.growRate
      this.ballArray[this.ballToGrow].displayHeight += gameOptions.growRate
    }
  }
}
```

## 两个球重叠时触发事件

```js
class PlayGame extends Phaser.Scene {
  create() {
    // 设置重叠事件
    this.physics.add.overlap(this.ballGroup, this.ballGroup, this.handleOverlap, null, this)
  }
  handleOverlap(ball1, ball2) {
    if (this.ballToGrow !== null && (ball1.index === this.ballToGrow || ball2.index === this.ballToGrow)) {
      // 相机拍照效果，不加也可
      this.cameras.main.flash()
      ball1.displayWidth = Math.max(ball1.displayWidth / 2, gameOptions.ballRadius)
      ball2.displayWidth = Math.max(ball2.displayWidth / 2, gameOptions.ballRadius)
      ball1.displayHeight = ball1.displayWidth
      ball2.displayHeight = ball2.displayWidth
      this.ballToGrow = null
    }
  }
}
```

至此，整个游戏我们就做完啦！🦍🦍🦍

> ***预览：<https://hewq.github.io/apps/a20200427growingcircle/index.html>***
>
> ***代码：<https://github.com/hewq/Phaser/tree/master/apps/a20200427growingcircle>***
>
> ***参考：<http://phaser.io/news/2020/04/hundreds-flash-game-prototype>***
>
> ***作者：<https://hewq.github.io/apps/resume/index.html>***
