# Phaser 小游戏——消消乐

| 需求 |
| --- |
| 1. 图标成矩形随机排列。 |
| 2. 可以拖动一行或一列。 |
| 3. 3个或以上个相同图案连在一起时会被消除。 |

*当前版本一次只能消除一处！*

*若有不足，欢迎指正！*

## 先把 `Phaser` 的主要代码搭起来

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Drag and Match</title>
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
            iconW: 100, // 图标宽度
            iconH: 100, // 图标高度
            rows: 10, // 行数
            columns: 7, // 列数
            iconNum: 6, // 图标数
            gameArea: { // 游戏区域
                x: 25,
                y: 300
            },
            movingX: 'x',
            movingY: 'y',
        }

        let gameAreaW = gameOptions.iconW * gameOptions.columns
        let gameAreaH = gameOptions.iconH * gameOptions.rows

        class PlayGame extends Phaser.Scene {
            constructor() {
                super('PlayGame')
            }
            preload() {
                this.load.spritesheet('sprite', 'sprite.png', {
                    frameWidth: gameOptions.iconW,
                    frameHeight: gameOptions.iconH
                })
            }
            create() {

            }
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
            scene: PlayGame
        }

        game = new Phaser.Game(gameConfig)
    </script>
</body>

</html>
```

## 设置游戏区域容器

```js
class PlayGame extends Phaser.Scene {
  create() {
    this.itemContainer = this.add.container(gameOptions.gameArea.x, gameOptions.gameArea.y)

    let mask = this.make.graphics()
    mask.fillStyle(0x00ffff, 1);
    mask.fillRect(gameOptions.gameArea.x, gameOptions.gameArea.y, gameOptions.iconW * gameOptions.columns, gameOptions.iconH * gameOptions.rows)
    this.itemContainer.setMask(new Phaser.Display.Masks.GeometryMask(this, mask))

    this.itemArr = [] // 用来存放所有的小球
  }
}
```

## 放置小球

```js
class PlayGame extends Phaser.Scene {
  create() {
    this.putSpirte()
  }
  putSpirte() {
    for (let i = 0; i < gameOptions.columns; i++) {
      this.itemArr[i] = []
      for (let j = 0; j < gameOptions.rows; j++) {
        let iconX = gameOptions.iconW * i
        let iconY = gameOptions.iconH * j
        let type = Phaser.Math.Between(0, gameOptions.iconNum - 1)
        let icon = this.add.sprite(iconX, iconY, 'sprite', type)
        icon.setOrigin(0, 0)
        icon.rows = j
        icon.columns = i
        icon.type = type

        this.itemArr[i][j] = icon
        this.itemContainer.add(icon)
      }
    }
  }
}
```

## 拖动小球

### 检测触碰区域获取触碰位置

```js
class PlayGame extends Phaser.Scene {
  checkArea(pointer) {
    let gameArea = new Phaser.Geom.Rectangle(gameOptions.gameArea.x, gameOptions.gameArea.y, gameOptions.columns * gameOptions.iconW, gameOptions.rows * gameOptions.iconH)
    return Phaser.Geom.Rectangle.Contains(gameArea, pointer.x, pointer.y)
  }
  getPos(pointer) {
    let col = (pointer.downX - gameOptions.gameArea.x) / gameOptions.iconW
    let row = (pointer.downY - gameOptions.gameArea.y) / gameOptions.iconH
    this.curRow = Math.floor(row)
    this.curCol = Math.floor(col)
  }
}
```

### 监听拖动

```js
class PlayGame extends Phaser.Scene {
  create() {
    this.input.on('pointerdown', this.startHandler, this)
    this.input.on('pointermove', this.moveHandler, this)
    this.input.on('pointerup', this.endHandler, this)
  }
  startHandler(pointer) {
    this.inArea = this.checkArea(pointer)
    if (this.inArea) {
      this.getPos(pointer)
      this.movingDir = null // 移动方向
      this.moveNum = 0 // 移动个数
      this.offset = 0 // 偏移量
      this.dir = 1 // 正负方向，1: 正方向， -1: 负方向
    }
  }
  moveHandler(pointer) {
    if (!this.inArea) return true

    let vector = new Phaser.Math.Vector2(pointer.x - pointer.downX, pointer.y - pointer.downY) // 移动向量

    this.movingItem = [] // 存放移动的小球
    if (this.movingDir) {
      if (this.movingDir === gameOptions.movingY) { // y 方向移动
        this.dir = vector.y > 0 ? 1 : -1 // 判断正负方向
        this.itemArr.map((col, colIndex) => {
          if (colIndex === this.curCol) {
            col.map((item, index) => {
              this.movingItem.push(item)
              item.y = (gameAreaH + (index * gameOptions.iconH + vector.y) % gameAreaH) % gameAreaH // 衔接滑动
            })
          }
        })

        this.offset = Math.abs(vector.y) % gameOptions.iconH
        this.moveNum = (Math.floor(Math.abs(vector.y) / gameOptions.iconH) + (this.offset < 50 ? 0 : 1)) % gameOptions.rows
      } else if (this.movingDir === gameOptions.movingX) { // x 方向移动
        this.itemArr.map(item => {
          this.movingItem.push(item[this.curRow])
        })
        this.dir = vector.x > 0 ? 1 : -1 // 判断正负方向
        this.itemArr.map((col, colIndex) => {
          col.map((item, index) => {
            if (index === this.curRow) {
              item.x = (gameAreaW + (colIndex * gameOptions.iconW + vector.x) % gameAreaW) % gameAreaW // 衔接滑动
            }
          })
        })
        this.offset = Math.abs(vector.x) % gameOptions.iconW
        this.moveNum = (Math.floor(Math.abs(vector.x) / gameOptions.iconW) + (this.offset < 50 ? 0 : 1)) % gameOptions.columns
      }
    } else { // 判断移动方向
      let angle = vector.angle()

      if (angle >= Math.PI / 4 && angle <= Math.PI * 3 / 4 || angle >= Math.PI * 5 / 4 && angle <= Math.PI * 7 / 4) {
        this.movingDir = gameOptions.movingY
      } else {
        this.movingDir = gameOptions.movingX
      }
    }
  }
  endHandler(pointer) {
    this.inArea = false
    if (this.movingDir === gameOptions.movingY) {
      this.tweens.add({
        targets: [...this.movingItem],
        duration: 200,
        y: (target, name, value) => {
          let ret = value + (this.offset > 50 ? this.dir * (gameOptions.iconH - this.offset) : -(this.dir * this.offset))
          if (Math.abs(ret) === gameAreaH) {
            this.overflowItem = target
          }
          return ret
        },
        onComplete: () => {
          this.overflowItem && (this.overflowItem.y %= gameAreaH)
        }
      })
    } else if (this.movingDir === gameOptions.movingX) {
      this.tweens.add({
        targets: [...this.movingItem],
        duration: 200,
        x: (target, name, value) => {
          let ret = value + (this.offset > 50 ? this.dir * (gameOptions.iconW - this.offset) : -(this.dir * this.offset))
          if (Math.abs(ret) === gameAreaW) {
            this.overflowItem = target
          }
          return ret
        },
        onComplete: () => {
          this.overflowItem && (this.overflowItem.x %= gameAreaW)
        }
      })
    }
  }
}
```

### 头/尾增加一个临时小球，让拖动看起来更顺畅

```js
class PlayGame extends Phaser.Scene {
  create() {
    this.tempItem = this.add.sprite(0, 0, 'sprite', 0).setAlpha(0)
    this.itemContainer.add(this.tempItem)
  }
  moveHandler(pointer) {
    if (this.movingDir) {
      if (this.movingDir === gameOptions.movingY) {
        this.itemArr.map((col, colIndex) => {
          if (colIndex === this.curCol) {
            if (this.dir > 0) {
              this.tempItem.setFrame(this.movingItem[this.movingItem.length - 1 - Math.floor(vector.y % gameAreaH / gameOptions.iconH)].type)
              this.tempItem.x = this.movingItem[this.movingItem.length - 1].x
              this.tempItem.y = vector.y % gameOptions.iconH - gameOptions.iconH
              this.tempItem.setAlpha(1)
              this.tempItem.setOrigin(0)
            } else if (this.dir < 0) {
              this.tempItem.setFrame(this.movingItem[Math.floor(Math.abs(vector.y % gameAreaH) / gameOptions.iconH)].type)
              this.tempItem.x = this.movingItem[0].x
              this.tempItem.y = vector.y % gameOptions.iconH
              this.tempItem.setAlpha(1)
              this.tempItem.setOrigin(0)
            }
          }
        })
      } else if (this.movingDir === gameOptions.movingX) {
        this.itemArr.map((col, colIndex) => {
          col.map((item, index) => {
            if (index === this.curRow) {
              if (this.dir > 0) {
                this.tempItem.setFrame(this.movingItem[this.movingItem.length - 1 - Math.floor(vector.x % gameAreaW / gameOptions.iconW)].type)
                this.tempItem.y = this.movingItem[0].y
                this.tempItem.x = vector.x % gameOptions.iconW - gameOptions.iconW
                this.tempItem.setAlpha(1)
                this.tempItem.setOrigin(0)
              } else if (this.dir < 0) {
                this.tempItem.setFrame(this.movingItem[Math.floor(Math.abs(vector.x) % gameAreaW / gameOptions.iconW)].type)
                this.tempItem.y = this.movingItem[0].y
                this.tempItem.x = vector.x % gameOptions.iconW
                this.tempItem.setAlpha(1)
                this.tempItem.setOrigin(0)
              }
            }
          })
        })
      }
    }
  }
  endHandler(pointer) {
    if (this.movingDir === gameOptions.movingY) {
      this.tweens.add({
        targets: [this.tempItem, ...this.movingItem],
        duration: 200,
        y: (target, name, value) => {
          let ret = value + (this.offset > 50 ? this.dir * (gameOptions.iconH - this.offset) : -(this.dir * this.offset))
          if (Math.abs(ret) === gameAreaH) {
            this.overflowItem = target
          }
          return ret
        },
        onComplete: () => {
          this.tempItem.setAlpha(0)
          this.overflowItem && (this.overflowItem.y %= gameAreaH)
        }
      })
    } else if (this.movingDir === gameOptions.movingX) {
      this.tweens.add({
        targets: [this.tempItem, ...this.movingItem],
        duration: 200,
        x: (target, name, value) => {
          let ret = value + (this.offset > 50 ? this.dir * (gameOptions.iconW - this.offset) : -(this.dir * this.offset))
          if (Math.abs(ret) === gameAreaW) {
            this.overflowItem = target
          }
          return ret
        },
        onComplete: () => {
          this.tempItem.setAlpha(0)
          this.overflowItem && (this.overflowItem.x %= gameAreaW)
        }
      })
    }
  }
}
```

### 移动后重置位置

```js
class PlayGame extends Phaser.Scene {
  endHandler(pointer) {
    this.resetPos()
  }
  resetPos() {
    if (this.movingDir === gameOptions.movingY) {
      let ahaedNum = (gameOptions.rows + this.dir * this.moveNum) % gameOptions.rows
      this.movingItem.map((item, index) => {
        item.columns = this.curCol
        item.rows = (index + ahaedNum) % gameOptions.rows
        this.itemArr[this.curCol][(index + ahaedNum) % gameOptions.rows] = item
      })
    } else if (this.movingDir === gameOptions.movingX) {
      let ahaedNum = (gameOptions.columns + this.dir * this.moveNum) % gameOptions.columns
      this.movingItem.map((item, index) => {
        item.columns = (index + ahaedNum) % gameOptions.columns
        item.rows = this.curRow
        this.itemArr[(index + ahaedNum) % gameOptions.columns][this.curRow] = item
      })
    }
  }
}
```

### 检测图案

```js
class PlayGame extends Phaser.Scene {
  endHandler(pointer) {
    this.traversal()
  }
  traversal() {
    this.traversalCol()
    this.traversalRow()
  }
  traversalCol() {
    let matching = false // 和上一个小球类型是否相同
    let lastType = -1 // 上一个小球的类型
    let matchArr = []
    let col
    let item
    // 判断列
    for (let colIndex in this.itemArr) {
      col = this.itemArr[colIndex]
      lastType = -1
      matchArr = []
      for (let index in col) {
        item = col[index]
        matchArr.push(item)
        if (lastType === item.type) {
          !matching && matchArr.unshift(col[index - 1])
          matching = true
          if (+index === gameOptions.rows - 1) {
            if (matchArr.length >= 3) {
              this.checkMatchCol(matchArr, col, colIndex)
              return
            }
          }
        } else {
          matchArr.pop()
          matching = false
          if (matchArr.length >= 3) {
            this.checkMatchCol(matchArr, col, colIndex)
            return
          }
          matchArr = []
        }
        lastType = item.type
      }
    }
  }
  traversalRow() {
    let lastType = -1
    let matchArr = []
    let matching = false
    let item
    for (let row = 0; row < gameOptions.rows; row++) {
      lastType = -1
      for (let col = 0; col < gameOptions.columns; col++) {
        item = this.itemArr[col][row]
        matchArr.push(item)
        if (lastType === item.type) {
          !matching && matchArr.unshift(this.itemArr[col - 1][row])
          matching = true
          if (row === gameOptions.rows - 1) {
            if (matchArr.length >= 3) {
              this.checkMatchRow(matchArr)
              return
            }
          }
        } else {
          matchArr.pop()
          matching = false
          if (matchArr.length >= 3) {
            this.checkMatchRow(matchArr)
            return
          }
          matchArr = []
        }
        lastType = item.type
      }
    }
  }
  checkMatchCol(arr, col, colIndex) {
    this.matchCol = true
    this.movedItemCol = []

    if (arr[0].rows !== 0) {
      let bottomRow = arr[arr.length - 1].rows
      for (let i = arr[0].rows; i > 0; i--) { // 下移的球
        this.movedItemCol.push(col[i - 1])
        this.itemArr[colIndex][bottomRow--] = col[i - 1]
        this.itemArr[colIndex][bottomRow + 1].columns = colIndex
        this.itemArr[colIndex][bottomRow + 1].rows = bottomRow + 1
      }
    }

    let len = arr.length
    for (let i = 0; i < arr.length; i++) { // 消失的球
      let iconX = parseInt(arr[0].columns) * 100
      let iconY = -1 * gameOptions.iconH * (i + 1)
      let type = Phaser.Math.Between(0, gameOptions.iconNum - 1)

      let icon = this.add.sprite(iconX, iconY, 'sprite', type)
      this.itemContainer.add(icon)
      icon.setOrigin(0, 0)
      icon.rows = --len
      icon.columns = colIndex
      icon.type = type

      this.itemArr[colIndex][icon.rows] = icon
      this.movedItemCol.push(icon)

      icon.x = iconX
      icon.y = iconY
    }
    len = arr.length
    this.tweens.add({
      targets: arr,
      duration: 400,
      alpha: 0,
      onComplete: () => {
        this.tweens.add({
          targets: this.movedItemCol,
          duration: 300,
          y: (target, name, value) => {
            return value + (len * 100)
          },
          onComplete: () => {
            this.traversal()
          }
        })
      }
    })
  }
  checkMatchRow(arr) {
    this.matchRow = true

    this.movedItemRow = []
    let rowIndex = arr[0].rows
    let bottomRow = rowIndex
    let range = parseInt(arr[0].columns) + parseInt(arr.length)
    for (let col = arr[0].columns; col < range; col++) {
      bottomRow = rowIndex
      for (let j = 0; j < rowIndex; j++) {
        this.itemArr[col][bottomRow] = this.itemArr[col][--bottomRow]
        this.itemArr[col][bottomRow].rows += 1
        this.movedItemRow.push(this.itemArr[col][bottomRow])
      }

      let iconX = this.itemArr[col][1].x
      let iconY = -100
      let type = Phaser.Math.Between(0, gameOptions.iconNum - 1)

      let icon = this.add.sprite(iconX, iconY, 'sprite', type)
      this.itemContainer.add(icon)
      icon.setOrigin(0, 0)
      icon.rows = 0
      icon.columns = col
      icon.type = type

      this.itemArr[col][0] = icon
      this.movedItemRow.push(icon)

      icon.x = iconX
      icon.y = iconY
    }

    this.tweens.add({
      targets: arr,
      duration: 400,
      alpha: 0,
      onComplete: () => {
        this.tweens.add({
          targets: this.movedItemRow,
          duration: 300,
          y: (target, name, value) => {
            return value + 100
          },
          onComplete: () => {
            this.traversal()
          }
        })
      }
    })
  }
}
```

### 每次只能消除一处

```js
class PlayGame extends Phaser.Scene {
  checkMatchCol(arr, col, colIndex) {
    if (this.matchRow) return
    this.matchCol = true

    this.tweens.add({
      targets: arr,
      duration: 400,
      alpha: 0,
      onComplete: () => {
        this.tweens.add({
          targets: this.movedItemCol,
          duration: 300,
          y: (target, name, value) => {
            return value + (len * 100)
          },
          onComplete: () => {
            this.matchCol = false
            this.traversal()
          }
        })
      }
    })
  }
  
  checkMatchRow(arr) {
    if (this.matchCol) return
    this.matchRow = true

    this.tweens.add({
      targets: arr,
      duration: 400,
      alpha: 0,
      onComplete: () => {
        this.tweens.add({
          targets: this.movedItemRow,
          duration: 300,
          y: (target, name, value) => {
            return value + 100
          },
          onComplete: () => {
            this.matchRow = false
            this.traversal()
          }
        })
      }
    })
  }
}
```

### 消除时不能移动

```js
class PlayGame extends Phaser.Scene {
  moveHandler(pointer) {
    if (this.matchRow || this.matchCol) return true
  }
}
```

### 初始化检测

```js
class PlayGame extends Phaser.Scene {
  putSprite(pointer) {
    this.traversal()
  }
}
```

至此，整个游戏我们就做完啦！🦍🦍🦍

> ***预览：<https://hewq.github.io/apps/a20200428match/index.html>***
>
> ***代码：<https://github.com/hewq/Phaser/tree/master/apps/a20200428match>***
>
> ***参考：<http://phaser.io/news/2020/04/hundreds-flash-game-prototype>***
>
> ***作者：<https://hewq.github.io/apps/resume/index.html>***
