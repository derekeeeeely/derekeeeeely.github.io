---
title: dz-music项目总结
tags:
  - react
  - mobx
  - parcel
abbrlink: 52ea43e5
date: 2018-01-19 13:04:54
---

&emsp;&emsp;一时兴起，写了个简单的音乐站，满足自己平时听歌的需要。欢迎朋友们试用、提[issue](https://github.com/derekeeeeely/dz-music/issues)、[star](https://github.com/derekeeeeely/dz-music)~
<!-- more -->

![](http://opo02jcsr.bkt.clouddn.com/0e03890a35c76d2eb7c7ec9c951bcd1e.png)


### 相关技术点

- view
  - react
  - antd
    平时做中后台业务习惯了，图个方便直接拿来用了，也不是很丑...
- store
  - mobx
    相比于redux更轻，代码侵入小，虽然没有中间件，但小应用也够用来管理前端状态了
- server
  - axios
    图个方便简单...后续封装个完善的请求模块
  - [NeteaseCloudMusicApi](https://github.com/Binaryify/NeteaseCloudMusicApi)
    在自己的server上clone了大佬的这个项目，跑起来以后为前端提供api，获取数据源
- 打包部署
  - [parcel](https://github.com/parcel-bundler/parcel)
  如官网所说，零配置，作为嫌弃webpack麻烦到死的我来说真的是够用了。
    - 优势
      自带热更新
      零配置
      速度快
    - 劣势
      热更新有问题
      没有sourceMap，调试很麻烦
  - nginx
    - 简单配置
  - steps
    - 本地build
    - 上传github repo
    - server上`git pull`

### 项目核心

&emsp;&emsp;由于audio和lyric是两个分开的组件，为了维护状态，这两个组件看起来都不是很独立。
&emsp;&emsp;后续考虑封装成一个更合理，更封闭的组件，比如对使用者来说只要传入musicList就可用。

#### audio组件

- 思路
  - 监听audio组件的事件
    举个栗子
    ```js
    // 实时进度条
    this.audio.addEventListener('timeupdate', () => {
      let progress = 0
      // 下一首的时候会先进到这里？！没获取到长度的时候会报错
      if (!!this.audio.duration) {
        progress = (this.audio.currentTime / this.audio.duration) * 100
      }
      this.props.passTime(this.audio.currentTime)
      this.props.form.setFieldsValue({
        slider: progress
      })
      this.setState({
        progress
      })
    })
    ```
    播放过程中监听时间变化，显示实时进度条、定位实时歌词位置。

- 使用
  - props
    - passTime (func)，用于将当前时间传出，存入store，方便lyric定位歌词位置
    - musicList，播放列表
    - insertMark，插入播放列表标志位 // 后续考虑优化
    - changeMark (func)，用于将当前歌曲在播放列表中位置传出，存入store，方便lyric获取歌词
  - 功能
    - 支持搜索歌曲，添加到播放列表
    - 支持播放歌曲、切换上下首、切换循环单曲和列表顺序模式
    - 支持播放中拖动进度条、显示当前时间、调节播放音量

#### lyric flow组件

- 思路
  - parse歌词字符串
    从网易云获取到的歌词长这样：
      ![](http://opo02jcsr.bkt.clouddn.com/cbae09421c6c201ec6a3e1bc7c084a00.png)
    我想要的是这样的：
      ![](http://opo02jcsr.bkt.clouddn.com/61b427ffe5b3fe9dcf6c5b3dd7b240d4.png)

    parse细节如下，其实也很简单，只要把对应歌词和时间对应上就好了

    ```js
    export default function parse(text) {
      const lines = text.split('\n')
      const pattern = /\[\d{2}:\d{2}.\d*\]/g
      const lyrics = []
      lines.map(line => {
        const timeLines = line.match(pattern)
        const lyricLine = line.replace(pattern, "")
        if(timeLines && timeLines.length) {
          for (let i=0; i<timeLines.length; i++) {
            const timeLine = timeLines[i]
            const min = timeLine.replace(/(\[|\:|\]|\.)/g, "").slice(0, 2)
            const sec = timeLine.replace(/(\[|\:|\]|\.)/g, "").slice(2, 4)
            const seconds = (+min) * 60 + (+sec)
            lyrics.push({
              [seconds]: lyricLine || '......'
            })
          }
        }
      })
      lyrics.sort((a, b) => {
        const keyA = +Object.keys(a)[0]
        const keyB = +Object.keys(b)[0]
        return keyA - keyB
      })
      return lyrics
    }
    ```
  - 监听播放时间，根据时间对应找到落在的歌词位置，显示旁边两条歌词

- 使用
  - props
    - lyric 当前歌曲的歌词
    - currentTime 当前播放时间
  - 功能
    - 现在是显示两条，后续考虑做成文字走马灯，高亮当前播放的一条

### 收获

#### 技术

- audio
  - 以前基本没用过，现在至少知道了一些生命周期事件以及常用属性的
- mobx
  - 和redux思想感觉是蛮接近的，都是单一数据源，都符合flux思想
  - 组件都需要去订阅store，mobx用inject，redux需要connect
  - redux把store搞成一个大块头，mobx把多个store分开
  - mobx直接通过action更新store，redux则通过action去调用reducer更新store
  - redux数据流的概念使得可以加入中间件，更好地处理异步请求
  - 小型应用用mobx绝对是方便快捷的
- 正则
  - 是个好东西，虽然以前基本不写，但感觉还是要好好学，学好将受益无穷啊
- willReceiveProps
  - 以前用得少，最近疯狂在用，是时候回顾下react生命周期了

#### 待挖掘

- react-router 4
- react 16
- parcel

这些东西都用上了，但是没有去具体了解，接下来会找个项目的机会好好研究下。
