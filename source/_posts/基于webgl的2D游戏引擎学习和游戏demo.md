---
title: 基于webgl的2D游戏引擎学习和游戏demo
date: 2018-07-30 02:12:12
tags:
- webgl
- javascript
categories:
- 2D游戏开发
---

​	这段时间在NUS，学习了关于计算机视觉的一些基础课程。选定的project是2D videogame，在此记录下关于利用webgl的2D游戏引擎制作过程和遇到的问题。游戏为[Escape](https://veeupup.github.io/Escape)，源码在 [github](https://github.com/Veeupup/Escape)上，下面是对游戏引擎的机制的分析和记录，便于理清思路并以后复习。

## 引擎结构分析

```
├─Engine				
│  ├─Cameras				相机类封装，决定截取viewport的那一部分
│  ├─Core					引擎核心，包括游戏初始化，绘制，游戏循环等部分
│  │  └─Resources			加载资源行为的封装
│  ├─GameObjects			游戏对象的封装（默认为刚体）
│  ├─Lights					渲染光源对象需要用到的光源	
│  ├─Particles				游戏物件
│  ├─Renderables			各种渲染类型的封装
│  ├─RigidShapes			刚体类的定义
│  ├─Shaders				和glsl文件进行交互的类
│  ├─Shadows				阴影类，在有光源时使用
│  └─Utils					各种
├─GLSLShaders				glsl文件，着色语言
├─lib						需要用到的数学库或者决定图形变化的库
└─MyGame					游戏主文件夹
    ├─Animation
    └─Objects
```

​	在此只是对引擎的主要文件夹的作用进行简要说明，具体文件还需要看源码。下面为具体分析。

<!-- more --> 

## 引擎工作原理

​	WebGL通过引入一个与可以在HTML5 的 canvas 元素中使用的OpenGL ES 2.0非常一致的API来实现这一点。 

​	它工作的速度很快的一个原因是，将视频和图片等处理交给GPU来处理，从而大大提高渲染速度，而且优点是可以和其他html元素混杂在一起使用，大大增加webgl的实用性。

## 完整过程分析

​	分析游戏引擎源码，这里举例记录一次基于游戏引擎完整的游戏场景的初始化到停止过程。这里举一个简单例子，主要是一整个场景的渲染流程，拿游戏的初始界面 Start.js 举例。

* 首先在 index.html 中的生成实例并通过游戏引擎调用

  ```javascript
  var dyeGame = new Start();
  gEngine.Core.initializeEngineCore('GLCanvas', dyeGame);
  ```

  这里首先生成 Start 对象，这里会进行 start对象的初始化，然后调用 引擎核心的 Core 的 initializeEngineCore() ，这里进行了整个引擎所需要组件的初始化，并在之后的步骤中不需要初始化。

  ```javascript
  var initializeEngineCore = function (htmlCanvasID, myGame) {
          _initializeWebGL(htmlCanvasID);
          gEngine.VertexBuffer.initialize();
          gEngine.Input.initialize(htmlCanvasID);
          gEngine.AudioClips.initAudioContext();
          gEngine.Physics.initialize();
          gEngine.LayerManager.initialize();
  
          // Inits DefaultResources, when done, invoke the anonymous function to call startScene(myGame).
          gEngine.DefaultResources.initialize(function () { startScene(myGame); });
      };
  ```

* 然后调用这个场景的 StartScene()函数，进入Start。

```javascript
scene.loadScene.call(scene); // Called in this way to keep correct context
gEngine.GameLoop.start(scene); // will wait until async loading is done and call scene.initialize()
```

这是这个函数的两个核心函数，分别是

1. **加载场景资源**
2. **开始游戏循环**

这两个过程非常重要，决定了整个游戏场景的渲染和行为。

* 在load scene之后，会进行 draw() 函数，也就是整个游戏场景的绘制，然后进入 Game Loop，这是游戏中行为检测的行为，这里用时间间隔来决定游戏每秒刷新的次数，也就是调用 update函数的频率，然后这里检测行为，这里就是整个游戏场景的运行。
* 在update中，有机制决定何时停止当前场景的Game Loop，然后退出当前的 Game loop，开始进行unloadscene函数，卸载掉我们之前loadscene加载的资源，为下一游戏场景的开启释放资源。
* 然后再次调用startscene，进入游戏设定的步骤。

### 过程总结

引擎初始化 initializeEngineCore =》传入scene ,startscene() =》加载资源 loadscene =》进行绘制draw() =》进入游戏循环 GameLoop() =》 循环并执行update() =》 游戏循环终止GameLoop.stop

=》卸载游戏资源unloadscene() =》开始新的startscene……



================================

================================

​	晚安，于2018/7/31  3:08 在NUS宿舍。