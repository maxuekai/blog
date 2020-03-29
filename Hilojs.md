# Hilojs 初体验

​	年前接了个需求，是在原本 `react` 的项目里，添加一个小游戏，鉴于后面的游戏需求将越来越复杂，准备使用游戏引擎进行开发，因此，正好趁这次机会了解、学习一下游戏引擎。

##  游戏引擎预研

1. [Egret](https://www.egret.com/) 、 [LayaAir](https://www.layabox.com/) 与 [Cocos](https://www.cocos.com/)  

   这三个都是很流行的游戏引擎，都有自己的 IDE 作为开发工具，支持 `Typescript` ，文档很详细，社区也很活跃，且都能跨平台开发

   缺点：游戏引擎都比较重，比较适合从头开始做，不适合在原有的项目里加进去，需要一定的学习成本

2. [Phaser](http://phaser.io/) 

   Phaser 在渲染方面直接封装了 Pixi，包含物理引擎，支持 `Typescript` ，适合做复杂的物理游戏，可以在原有项目里引入

   缺点：包太大，差不多`1M`

3. [Hilojs](http://hiloteam.github.io/index.html)

   阿里团队推出的一个开源项目，支持模块化开发，适合做营销小游戏，已有多个阿里活动项目使用，官方文档详细，体积小，整个包大小只有 `70k` 左右，支持 `Typescript`

   缺点：虽然支持 Typescript，但描述文件不完整，在 ts 环境下会有较多报错

​    最后我们决定选择 `Hilojs` 作为我们的入门砖；`Hilojs` 在性能体验方面也很强，我们借用官方给的两个 demo，`flappy bird` 和 `fruit ninja` 在  `iphone 4s（ios 9）` 和 `华为 Honor PLK-UL00（EMUI 系统 4.0.3）`  下测试，都可以很流畅地运行。

##  Hilojs 之初体验

​	不得不说，Hilojs 确实是个很优秀的游戏引擎，它为我们封装了很多常用的操作：资源加载、音频播放、碰撞检测等，我们都可以很快速地实现；在开发过程中，也遇到了一些小坑...

#### Hilojs 概览

​	和其他游戏引擎一样，Hilojs 有 Stage 舞台（场景）作为可视对象根节点，其他游戏元素（sprite、bitmap、text）作为子节点在场景中渲染，并在子节点中添加动画、操作来实现游戏交互，最终生成一个完整的游戏；Hilojs 集合了 tween 动画库，可以使用 tweenjs 实现节点动画；与其他游戏引擎不同的是，Hilojs 只能生成一个 Stage 舞台，你可以在一个 Stage 舞台上任意切换 Container，来达到切换场景的作用，但是这可能与 Hilojs 的开发初衷有所违背，Hilojs 本来就是阿里用于开发大促使用的小游戏，并不需要过多的舞台（场景），而且这也是它能更容易引入项目的原因吧。

#### Hilojs 开发

​	Hilojs 的 api 与开发在官网写的很详细，也有 `flappy bird` 的代码可做参考，理解起来也不难，主要流程就是：

 `加载资源（图片、音频）` —> `初始化舞台，设置定时器不断刷新舞台` —> `在舞台里添加游戏元素`  —> `增加游戏逻辑判断`

##### 1. 加载资源

​	Hilojs 提供 `LoadQueue` 类来管理资源（主要是图片资源）

```typescript
const Assets = Hilo.Class.create({
  Mixes: Hilo.EventMixin,
  
  constructor: function () {
    this.queue = new Hilo.LoadQueue();
  },
  
  /** 添加图片资源 */
  addImage: function (imageUrl) {
    this.queue.add({ src: imageUrl });
  },
  
  /** 开始加载资源 */
  load: function () {
    console.log('开始加载资源');
    this.queue.on('complete', () => {
      console.log('资源加载完毕');
      this.queue.off('complete');
      this.fire('complete');
    });
    this.queue.start();
  }
});
```

​	游戏中音频资源也是很常见的，但 `LoadQueue` 类不能直接加载音频资源，所以我们需要另辟蹊径，找其他方法加载音频资源。我使用的是 `Hilo.WebSound` 来管理音频，`Hilo.WebSound` 提供一个静态方法 `getAudio` 来获取音频实例，从而达到控制音频的功能

```typescript
const Bgm = Hilo.Class.create({
  Mixes: Hilo.EventMixin,
  
  constructor: function (props) {
    this.bgmUrls = props.bgms;
    this.bgmUrls.forEach((bgm, index) => {
      // 获取音频实例
      this.bgm[index] = Hilo.WebSound.getAudio({ src: bgm });
      this.bgm[index].on('load', this.complete.bind(this));
    });
  },
  
  /** bgm 加载完成 */
  complete: function() {
    // 没加载完成一个 bgm，loaded ++
    ++this.loaded;
    // 当 loaded >= bgm 总数时，则为全部 bgm 加载完成，此时促发外部 complete 事件
    if (this.loaded >= Object.keys(this.bgm).length) {
      this.fire('complete');
    }
  },
  
  /** 开始加载 bgm */
  load: function() {
    this.bgm.forEach(bgm => {
      if (bgm.loaded) {
        this.complete();
      } else {
        bgm.load();
      }
    });
  }
});
```

##### 2. 初始化舞台

​	舞台是 Hilojs 最基础的部分，其他所有需要展示的元素，都是需要添加到舞台上才能被用户所看见，因此加载完资源之后，便需要初始化舞台；舞台类似于 `HTML` 里的 <body> 标签，由于我们的游戏主要是在手机端上使用的，元素大小自适应是非做不可了，由于开始做的时候，游戏的设计稿与切图已经在我们手里了，为了之后方便定位元素大小，我们已设计稿的宽度为标准，通过缩放舞台的大小，来达到所有元素自适应大小的效果

```typescript
class Game {
  /** 初始化舞台 */
  private initStage = () => {
    // 缩放倍数
    const scale = width / GameConfig.view.width;
    this.stage = new Hilo.Stage({
      container: 'game',
      width: GameConfig.view.width,
      height: height * GameConfig.view.width / width,
      scaleX: scale,
      scaleY: scale,
    });
    
    // 启动计时器
    this.ticker = new Hilo.Ticker(60);
    this.ticker.addTick(this.stage);
    this.ticker.start(true);
    
    // 绑定交互事件
    this.stage.enableDOMEvent([Hilo.event.POINTER_START, Hilo.event.POINTER_MOVE, Hilo.event.POINTER_END]);
    
    // 添加其他游戏元素
    this.initBackground();
    ...
    
    // 更新舞台
    this.stage.onUpdate = this.onUpdate;
  }
}
```

##### 3. 添加游戏元素

​	舞台建立好了后，我们就需要往舞台里添加游戏元素；Hilojs 提供一个 `View` 类当作基类，并衍生出其他常用的游戏元素类型，比如：`Bitmap（图片）`、`Graphics（形状）`、`Sprite（精灵）`、`Text（文本）` 、`Container（容器）` 等，舞台 `Stage` 也是继承自 `View` 类的。

​	游戏界面一般会分为几个大部分，像游戏主体、计时器等，我们一般会把每个大部分用一个 `Container` 的包裹起来当作一个整体来使用，就像 HTML 里 <div> 标签里会有 <p> 标签、<a> 标签等

```typescript
// 游戏状态栏
const Toolbar = Hilo.Class.create({
  Extends: Hilo.Container,
  
  /** 初始化计时器 */
  initCountDown: function () {
    this.countdown = new Hilo.Text({ text: 30 }).
    	setFont('40px Roboto, Arial')
      .addTo(this);
  },
  
  /** 更新倒计时 */
  updateDuration: function(duration: number) {
    this.countdown.text = this.handleCountDown(duration);
  },
  
  /** 初始化计分器 */
  initScoreBar: function () {
    this.scoreText = new Hilo.Text({ text: 0 })
      .setFont('50px Roboto, Arial')
      .addTo(this);
  },
  
  /** 更新分数 */
  updateScore: function(text: string) {
    this.scoreText.text = text;
  },
});
```

```typescript
class Game {
  /** 初始化状态栏 */
  private initToolBar = () => {
    this.toolbar = new ToolBar().addTo(this.stage);
  }
}
```

##### 4. 游戏逻辑

​	`View` 类提供 `hitTestObject` 与 `hitTestPoint`  来做碰撞检测，只限于多边形（圆形）的碰撞检测，由于图片往往是不规则的图形，我们的游戏也不需要如此精确的碰撞，因此，可以设置碰撞物体的 `bonusArea` 来设置物体的点，以此作为碰撞检测的点

```typescript
/** 可碰撞点连成的容器多边形 */
const bodyBonus = [
  {x: 5, y: 20},
  {x: 55, y: 8},
  {x: 330, y: 8},
  {x: 375, y: 20},
  {x: 355, y: 115},
  {x: 280, y: 158},
  {x: 280, y: 190},
  {x: 115, y: 190},
  {x: 115, y: 158},
  {x: 34, y: 115},
  {x: 5, y: 20}
];

const Container = Hilo.Class.create({
  ...
  init: function () {
    this.propContainer = new Hilo.Container({
      id: 'propContainer',
      width: GameConfig.view.width,
      // 设置容器的点
      boundsArea: bodyBonus
    }).addTo(this);
  }
});
```

#### Hilojs 开发遇到的问题

Hilojs 帮我们封装了很多游戏开发需要用到的方法，但在开发过程中，也遇到了一些问题；

 1. typescript 声明文件

    Hilojs 最近一次更新是在 10 months 之前，但是 `d.ts` 声明文件更新是在 2 years 之前，声明文件跟 API 不能完全匹配，导致使用 typescript 开发的时候，会有很多类型问题，最方便的就是使用 `as any` 解决，但也不是很友好；

 2. `WebAudio` 的一个 bug

    设置 `WebAudio` 的 `loop` 为 `true` 时，使用 `.stop` 的方法无法使其停止下来，要先将 `loop = false` 之后，才可使用 `stop`

 3. `Graphics` 导致帧率下降

    安卓上使用 `Hilojs` 的 `Graphics` 类的 `drawRoundRectComplex` 方法，会导致帧率下降； 具体 `drawRoundRectComplex` 方法导致帧率下降的原因，猜测与 `arc` 绘画圆角有关，试过使用 `Graphics` 绘制其他形状，只要不涉及圆角的都不会使帧率下降

#### 总结

​	`Hilojs` 确实是一个很适合前端入门的优秀的游戏引擎，相比于其他游戏引擎来说，很符合前端开发的习惯；只是最近一次更新也是好几个月前了，真心希望能保持更新，也把 `typescript` 支持得更好。













