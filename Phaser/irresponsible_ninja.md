# Phaser 小游戏 —— 不靠谱的忍者

| 需求 |
| --- |
| 1. 按住屏幕，棍子伸长，放开手指，棍子放下。 |
| 2. 具有计时功能。 |
| 3. 计时结束或者棍子没有放到安全区域，则游戏结束。 |

## 单独一个 `Scene` 作为背景，防止显示 `Scene` 重叠时有遮挡

```typescript
// background.ts
export default class extends Phaser.Scene {
    constructor() {
        super({
            key: 'BackgroundScene',
            active: true
        });
    }

    create(): void {
        const graphics = this.add.graphics();

        graphics.fillGradientStyle(0x241a47, 0x241a47, 0x274aa0, 0x274aa0);
        graphics.fillRect(0, 0, window.game.width, window.game.height);

        this.scene.launch('FootScene');
        this.scene.launch('StartScene');
    }
}
```

## 底部组件

### 底部的云层效果作为公共组件使用

```typescript
// foot.ts
import pngCloud from '@images/cloud.png';

const config: FootConfig = {
    circleNum: 7,
    blueCircleFrame: 0,
    whileCircleFrame: 1,
    blueCloudY: 160,
    whileCloudY: 190,
    height: 90
};

export default class extends Phaser.Scene {
    constructor() {
        super({
            key: 'FootScene'
        });
    }

    preload(): void {
        this.load.spritesheet('ssCloud', pngCloud, { frameWidth: 256, frameHeight: 256});
    }

    create(): void {

        const blueGroup = this.add.group([], {
            key: 'ssCloud',
            frame: [config.blueCircleFrame],
            frameQuantity: config.circleNum,
            setXY: {
                y: config.blueCloudY,
                stepX: 120,
                stepY: 0
            }
        });

        const whileGroup = this.add.group([], {
            key: 'ssCloud',
            frame: [config.whileCircleFrame],
            frameQuantity: config.circleNum,
            setXY: {
                y: config.whileCloudY,
                stepX: 120,
                stepY: 0
            }
        });

        this.resize();

        window.addEventListener('resize', () => {
            this.resize();
        });
    }

    resize(): void {
        const viewHeight = document.documentElement.clientHeight / window.rem;
        const camerasY = (window.game.height - viewHeight) / 2 + viewHeight - config.height;
        this.cameras.main.setPosition(0, camerasY);
    }
}
```

### 添加动画

```typescript
// foot.ts
export default class extends Phaser.Scene {
    create(): void {
        this.add.tween({
            targets: blueGroup.getChildren(),
            props: {
                y: (target) => {
                    return target.y + Phaser.Math.Between(-5, 0);
                },
                scale: (target) => {
                    return target.scale + Phaser.Math.FloatBetween(-0.05, 0.05);
                }
            },
            duration: 3000,
            yoyo: true,
            repeat: -1,
            ease: Phaser.Math.Easing.Sine.InOut
        });

        this.add.tween({
            targets: whileGroup.getChildren(),
            props: {
                y: (target) => {
                    return target.y + Phaser.Math.Between(-5, 0);
                },
                scale: (target) => {
                    return target.scale + Phaser.Math.FloatBetween(0, 0.1);
                }
            },
            duration: 3000,
            yoyo: true,
            repeat: -1,
            ease: Phaser.Math.Easing.Sine.InOut
        });
    }
}
```

### 适配，始终显示在窗口底部，css 的 fixed 效果

```typescript
// foot.ts
export default class extends Phaser.Scene {
    create(): void {
        this.resize();

        window.addEventListener('resize', () => {
            this.resize();
        });
    }

    resize(): void {
        const viewHeight = document.documentElement.clientHeight / window.rem;
        const camerasY = (window.game.height - viewHeight) / 2 + viewHeight - config.height;
        this.cameras.main.setPosition(0, camerasY);
    }
}
```

## 开始页面

```typescript
// start.ts
import pngBtnStart from '@images/btn_start.png';
import pngTitle from '@images/title.png';

export default class extends Phaser.Scene {
    constructor() {
        super({
            key: 'StartScene'
        });
    }

    preload(): void {
        this.load.image('imgBtnStart', pngBtnStart);
        this.load.image('imgTitle', pngTitle);
    }

    create(): void {
        this.add.rectangle(window.game.width / 2, window.game.height / 2, window.game.width, window.game.height, 0x000000, 0.5);

        this.add.image(window.game.width / 2, 240, 'imgTitle').setOrigin(0.5, 0);

        const btnStart = this.add.sprite(window.game.width / 2, window.game.height / 2, 'imgBtnStart').setInteractive();

        btnStart.on('pointerdown', () => {
            this.scene.start('MainScene');
        });

        this.add.tween({
            targets: btnStart,
            props: {
                y: (target) => {
                    return target.y + 20;
                }
            },
            yoyo: true,
            loop: -1,
            duration: 2000,
            ease: Phaser.Math.Easing.Sine.InOut
        });
    }
}
```

## 主场景

### 显示静态信息

```typescript
// main.ts
import pngTips from '@images/tips.png';
import pngProcess from '@images/process_border.png';
import pngSprites from '@images/sprites.png';
import pngNinja from '@images/ninja.png';

let txtDistance: Phaser.GameObjects.Text; // 文本，显示距离
let rect: Phaser.GameObjects.Rectangle; // 进度条
let stick: Phaser.GameObjects.Rectangle; // 棍子

let processTimerEvent: Phaser.Time.TimerEvent; // 计时事件

let overContainer: Phaser.GameObjects.Container; // 游戏结束内容容器
let gameContainer: Phaser.GameObjects.Container; // 游戏内容容器

let prePlatformDistance: number; // 到上一个站台的距离
let nextPlatformDistance: number; // 到下一个站台的距离
let curPlatformWidth: number; // 当前站台宽度
let prePlatformWidth: number; // 上一个站台宽度
let nextPlatformWidth: number; // 下一个站台宽度

let curPlatformX: number; // 当前站台位置
let nextPlatformX: number; // 下一个站台位置

let ninja: Phaser.GameObjects.Sprite; // 不靠谱的忍者本者

let distance = 0; // 距离，站台数
let isPlaying = false; // 是否处于处理流程中，区间在从按下到释放后的动画播放结束
let isStart = false; // 是否开始游戏

const config: GameConfig = {
    processLen: 500, // 进度条长度
    processHeight: 29, // 进度条高度
    platformHeight: 600, // 站台高度
    stickWidth: 10, // 棍子宽度
    stickHeight: -10 // 棍子长度，坐标系原因，取负值
};

// 变化的值单独拧出来
let stickHeight = config.stickHeight;
let processLen = config.processLen;

export default class extends Phaser.Scene {
    constructor() {
        super({
            key: 'MainScene'
        });
    }

    preload(): void {
        this.load.image('imgTips', pngTips);
        this.load.image('imgProcess', pngProcess);
        this.load.spritesheet('ssSprites', pngSprites, { frameWidth: 150, frameHeight: 150});
        this.load.spritesheet('ssNinja', pngNinja, { frameWidth: 462 / 6, frameHeight: 388 / 4});
    }

    create(): void {
        // 提示信息
        const tips = this.add.image(window.game.width / 2, 400, 'imgTips');

        // 进度条
        const process = this.add.image(0, 0, 'imgProcess');
        txtDistance = this.add.text(260, -24, 'DISTANCE: 0', {
            fontFamily: 'Arial',
            fontSize: 40,
            color: '#ffffff'
        }).setOrigin(1);

        rect = this.add.rectangle(-250, 0, config.processLen, config.processHeight, 0xffffff).setOrigin(0, 0.5);

        // 进度条区域内容容器
        const timerContainer = this.add.container(window.game.width / 2, 400, [process, txtDistance, rect]);
        timerContainer.setAlpha(0);

        // 游戏结束内容
        const btnPlay = this.add.sprite(-110, 0, 'ssSprites', 0).setInteractive();
        const btnHome = this.add.sprite(110, 0, 'ssSprites', 1).setInteractive();
        overContainer = this.add.container(window.game.width / 2, 1800, [btnPlay, btnHome]);
        overContainer.setAlpha(0.5);

        // 初始化忍者
        ninja = this.add.sprite(90, -600, 'ssNinja', 0).setOrigin(1);

        this.anims.create({
            key: 'stand',
            frames: this.anims.generateFrameNumbers('ssNinja', { start: 0, end: 11 }),
            frameRate: 12,
            repeat: -1,
            yoyo: true
        });

        this.anims.create({
            key: 'walk',
            frames: this.anims.generateFrameNumbers('ssNinja', { start: 12, end: 19 }),
            frameRate: 12,
            repeat: -1
        });

        ninja.play('stand');

        // 初始化棍子
        stick = this.add.rectangle(90, -config.platformHeight, config.stickWidth, config.stickHeight, 0x000000).setOrigin(1, 0);

        gameContainer = this.add.container(window.game.width / 2, window.game.height, [ninja, stick]);
        gameContainer.setSize(window.game.width, window.game.height);
        gameContainer.setPosition(0, window.game.height);

        // 初始化站台
        const firstPlatform = this.add.rectangle(0, 0, 100, config.platformHeight, 0x000000).setOrigin(0, 1);
        gameContainer.add(firstPlatform);
        gameContainer.bringToTop(ninja);
        curPlatformX = 0;
        curPlatformWidth = 100;
        prePlatformDistance = 0;
    }
}
```

### 生成站台

```typescript
// main.ts
export default class extends Phaser.Scene {
    create(): void {
        this.createPlatform(false);
    }

    createPlatform(playAnim: boolean): void {
        nextPlatformDistance = Phaser.Math.Between(150, 300); // 随机下个站台的距离
        nextPlatformWidth = Phaser.Math.Between(80, 150); // 随机下个站台的宽度

        nextPlatformX = curPlatformX + nextPlatformDistance + curPlatformWidth; // 计算下个站台的位置

        const rect = this.add.rectangle(nextPlatformX + 750, 0, nextPlatformWidth, config.platformHeight, 0x000000).setOrigin(0, 1); // 添加站台

        gameContainer.add(rect);
        gameContainer.bringToTop(ninja); // 忍者提到最上层，掉下去的时候不会被站台遮挡

        if (playAnim) { // 是否播放站台出现时的动画
            this.add.tween({ // 游戏容器移动
                targets: gameContainer,
                props: {
                    x: (target) => {
                        return target.x - (prePlatformDistance + prePlatformWidth);
                    }
                },
                duration: 300,
                ease: Phaser.Math.Easing.Sine.In
            });

            this.add.tween({ // 新增站台移动
                targets: rect,
                props: {
                    x: nextPlatformX
                },
                duration: 500,
                ease: Phaser.Math.Easing.Sine.In
            });

            this.add.tween({ // 忍者回到指定位置
                targets: ninja,
                props: {
                    y: (target) => {
                        return target.y + 10;
                    },
                    x: curPlatformX + (curPlatformWidth - 20)
                },
                duration: 300
            });

            this.add.tween({ // 棍子回到指定位置
                targets: stick,
                props: {
                    alpha: 0
                },
                duration: 300,
                onComplete: () => {
                    stick.setSize(config.stickWidth, config.stickHeight);
                    stick.setAngle(0);
                    stick.setX(curPlatformX + (curPlatformWidth - 10));
                    stick.setAlpha(1);
                    isPlaying = false;
                }
            });
        } else {
            rect.setPosition(nextPlatformX, 0);
        }
    }
}
```

### 生成动画

```typescript
// main.ts
createWalkTimeline(): void {
    const walkTimeline = this.tweens.createTimeline();

    // 棍子动画
    walkTimeline.add({
        targets: stick,
        props: {
            angle: 90
        },
        duration: 500,
        ease: Phaser.Math.Easing.Bounce.Out,
        onComplete() {
            ninja.play('walk');
        }
    });

    // 走路动画
    walkTimeline.add({
        targets: ninja,
        props: {
            x: (target) => {
                return target.x + Math.abs(stickHeight) + ninja.getBounds().width / 2;
            },
            y: (target) => {
                return target.y - 10;
            }
        },
        duration: 500,
        onComplete: () => {
            ninja.play('stand');
            this.calcDistance();
            walkTimeline.destroy();
        }
    });

    walkTimeline.play();
}
```

### 计算一次游戏结果

```typescript
// main.ts
calcDistance(): void {
    const near = Phaser.Math.Distance.Between(stick.x, 0, nextPlatformX, 0); // 下个站台的近端
    const far = Phaser.Math.Distance.Between(stick.x, 0, nextPlatformX + nextPlatformWidth, 0);// 下个站台的远端

    // 当前值变化
    curPlatformX = nextPlatformX;
    prePlatformWidth = curPlatformWidth;
    curPlatformWidth = nextPlatformWidth;
    prePlatformDistance = nextPlatformDistance;

    if (-stickHeight > near && -stickHeight < far) { // 安全区域
        this.createPlatform(true);
        txtDistance.setText(`DISTANCE: ${++distance}`); // 距离 +1
    } else {
        this.gameover(); // 游戏结束
        if (-stickHeight < near) {
            this.add.tween({
                targets: stick,
                props: {
                    angle: 180
                },
                duration: 800,
                ease: Phaser.Math.Easing.Bounce.Out
            });
        }
    }

    // 重置棍子长度
    stickHeight = config.stickHeight;
}
```

### 监听事件

```typescript
// main.ts
export default class extends Phaser.Scene {
    create(): void {
        let stickTimerEvent: Phaser.Time.TimerEvent;

        this.input.on('pointerdown', () => { // 点击屏幕
            if (!isPlaying) {
                tips.setAlpha(0);
                timerContainer.setAlpha(1);
                // 开始计时
                if (!isStart) {
                    isStart = true;
                    processTimerEvent = this.time.addEvent({
                        callback: this.processTimer.bind(this),
                        loop: true,
                        delay: 1000
                    });
                }

                // 棍子伸长事件
                stickTimerEvent = this.time.addEvent({
                    callback: this.stickTimer,
                    loop: true,
                    delay: 10
                });
            }

        });

        this.input.on('pointerup', () => { // 释放
            if (stickTimerEvent && !isPlaying) {
                isPlaying = true;
                stickTimerEvent.destroy();
                this.createWalkTimeline();
            }
        });

        // 重新开始
        btnPlay.on('pointerdown', (pointer, localX, localY, event) => {
            event.stopPropagation(); // 阻止冒泡
            this.reset();
            this.scene.restart();
        });

        // 回到首页
        btnHome.on('pointerdown', (pointer, localX, localY, event) => {
            event.stopPropagation();
            this.reset();
            this.scene.start('StartScene');
        });
    }

    stickTimer(): void {
        stickHeight -= 25;
        stick.setSize(config.stickWidth, stickHeight);
    }

    processTimer(): void {
        processLen -= 5;
        rect.setSize(processLen, config.processHeight);
        if (processLen === 0) {
            processTimerEvent.destroy();

            this.gameover();
        }
    }

    // 数据复位
    reset(): void {
        processLen = config.processLen;
        isPlaying = false;
        isStart = false;
        distance = 0;
    }
}
```

### 游戏结束

```typescript
// main.ts

gameover(): void {
    processTimerEvent.destroy();
    // shake
    const vec2 = new Phaser.Math.Vector2(0.005, 0.01);
    this.add.tween({
        targets: ninja,
        ease: Phaser.Math.Easing.Linear,
        duration: 600,
        props: {
            angle: 45,
            x: (target) => {
                return target.x + 50;
            },
            y: 50
        },
        onComplete: () => {
            // 晃动效果
            this.cameras.main.shake(200, vec2);
            this.scene.get('FootScene').cameras.main.shake(200, vec2);
            this.scene.get('BackgroundScene').cameras.main.shake(200, vec2);
        }
    });

    this.add.tween({
        targets: overContainer,
        props: {
            y: window.game.height / 2,
            alpha: 1
        },
        delay: 1200,
        duration: 800,
        ease: Phaser.Math.Easing.Back.InOut
    });

    this.add.tween({
        targets: gameContainer,
        props: {
            alpha: 0
        },
        delay: 1000,
        duration: 800,
        ease: Phaser.Math.Easing.Back.InOut
    });
}

```

## 写在后面

1. phaser 本身的适配似乎没有相对窗口定位的功能（类似 css 的 fixed），如果游戏中有这样的需求的话，就得自己手动再做多一步适配。
2. phaser 和 webpack 结合导入资源时，感觉有些麻烦，需要先 `import` 之后再使用 phaser 的 `loader`，不能一步到位。

以上若有好的处理方法，还请各位不吝赐教。

> ***预览：<https://hewq.github.io/apps/a20201112_irresponsible_ninja/index.html>***
>
> ***代码：<https://github.com/hewq/Phaser/tree/master/apps/a20201112_irresponsible_ninja>***
>
> ***参考：<https://triqui.itch.io/irresponsible-ninja>***
>
> ***作者：<https://hewq.github.io/apps/resume/index.html>***
