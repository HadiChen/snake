<template>
  <div class="wrap" ref="wrap">
    <div class="main">
      <audio autoplay preload="auto" ref="bgm" loop class="bgm">
        <source src="../assets/music/bgm.mp3" type="audio/mpeg"/>
      </audio>
      <div
        class="map"
        :style="{
          'paddingTop': remainTB + 'px',
          'paddingBottom': remainTB + 'px',
          'paddingLeft': remainLR + 'px',
          'paddingRight': remainLR + 'px'
        }">
        <div
          class="map-row"
          v-for="row in mapLists"
          :key="row.key">
          <div
            class="map-col"
            v-for="col in row.colList"
            :key="col.key">
            <div
              class="snake-part"
              :class="[{
                'header': snake.indexOf(col.key) === 0,
                'body':
                  snake.indexOf(col.key) > -1 &&
                  snake.indexOf(col.key) !== 0 &&
                  snake.indexOf(col.key) !== snake.length - 1,
                'footer':
                  snake.indexOf(col.key) === snake.length - 1,
                'egg': col.isEgg
              }, dirClassName]"></div>
          </div>
        </div>

        <transition name="fade">
          <div class="pop-ups" v-show="showCountdown">
            <h1 class="countdown-wrap__text">{{ countdown }}</h1>
          </div>
        </transition>

        <transition name="fade">
          <div class="pop-ups" v-show="isShowPop">
            <div class="pop-ups__content">
              <p class="title">
                游戏结束
              </p>
              <p class="desc">本次得分为：{{ getScoreCount }}</p>
              <img class="code-img" src="../assets/images/code.jpg" alt="">
              <p class="desc f14">长按关注作者公众号了解一下</p>
              <p class="desc f14">作者：黄蕴宇</p>
              <p class="desc f14">广州市工贸技师学院</p>
              <p class="desc f14">网站维护与开发</p>
              <button type="button" class="reStart" @click="reStart">
                重新开始
              </button>
            </div>
          </div>
        </transition>
      </div>
    </div>
  </div>
</template>

<script>
import Hammer from 'hammerjs'

export default {
  data () {
    return {
      isShowPop: false,
      showCountdown: false,
      countdown: 3,
      // 剩余上下
      remainTB: 0,
      // 剩余左右
      remainLR: 0,
      // 行数量
      rowNumber: 0,
      // 列数量
      columnNumber: 0,
      mapLists: [],
      // 位置坐标
      x: 0,
      y: 0,
      // 吃蛋数量
      scoreCount: 0,
      // 蛋位置
      eggX: 0,
      eggY: 0,
      // 方向
      direction: 'up',
      // 避免同时多次改变方向句柄
      changeDir: true,
      // 运动时间句柄
      moveTimer: null,
      snake: [],
      // 蛇的长度
      snakeLen: 3,
      // 速度
      speed: 300
    }
  },
  computed: {
    getScoreCount () {
      return this.scoreCount * 10
    },
    dirClassName () {
      switch (this.direction) {
        case 'up':
          return ''
        case 'down':
          return 'rotate180'
        case 'left':
          return 'rotate270'
        case 'right':
          return 'rotate90'
      }
    }
  },
  methods: {
    // 设置地图信息
    setMapData () {
      for (let i = 0; i < this.rowNumber; i++) {
        let obj = {
          key: i,
          colList: []
        }
        for (let k = 0; k < this.columnNumber; k++) {
          let c = {
            key: `${i}-${k}`,
            isEgg: false
          }
          this.$set(obj.colList, k, c)
        }
        this.$set(this.mapLists, i, obj)
      }
    },
    // 随机数
    random (min, max) {
      return Math.floor(Math.random() * (max - min + 1) + min)
    },
    // 生成蛋
    createNewEgg () {
      // 随机出新的egg的下标的x和y值
      this.eggX = this.random(0, this.rowNumber - 1)
      this.eggY = this.random(0, this.columnNumber - 1)

      if (this.snake.indexOf(this.mapLists[this.eggX].colList[this.eggY].key) > -1 || this.mapLists[this.eggX].colList[this.eggY].isEgg) {
        this.createNewEgg()
      } else {
        this.mapLists[this.eggX].colList[this.eggY].isEgg = true
      }
    },
    // 计算前进方向
    compCoordinate (direction, x, y) {
      let x1 = x
      let y1 = y
      switch (direction) {
        case 'left':
          --y1
          break
        case 'right':
          ++y1
          break
        case 'up':
          --x1
          break
        case 'down':
          ++x1
          break
        default:
          return
      }
      return {
        x: x1,
        y: y1
      }
    },
    // 蛇运动
    snakeMove () {
      // 根据上面设置的方向来设置蛇头的位置
      switch (this.direction) {
        case 'left':
          --this.y
          break
        case 'right':
          ++this.y
          break
        case 'up':
          --this.x
          break
        case 'down':
          ++this.x
          break
      }
      // 是否撞墙了
      if (this.x < 0 || this.y < 0 || this.x >= this.rowNumber || this.y >= this.columnNumber) {
        this.isShowPop = true
        clearInterval(this.moveTimer)
        return
      }
      // 是否吃到自己
      if (this.snake.indexOf(this.mapLists[this.x].colList[this.y].key) > -1) {
        this.isShowPop = true
        clearInterval(this.moveTimer)
        return
      }
      // 吃到蛋了
      if (this.eggX === this.x && this.eggY === this.y) {
        this.mapLists[this.x].colList[this.y].isEgg = false
        ++this.scoreCount
        ++this.snakeLen
        this.createNewEgg()
      }
      // 前进
      this.snake.unshift(this.mapLists[this.x].colList[this.y].key)
      // 保持🐍的长度
      if (this.snakeLen < this.snake.length) {
        this.snake.pop()
      }
    },
    setDirection (direction) {
      // 先判断是否需要改变方向,true表示需要,false表示不需要
      if (!this.changeDir && !direction) return
      // 为了合理处理蛇的移动,需要判断蛇头和蛇身
      // 假设蛇向右移动,点方向键左,右键都不需要做出响应
      if (this.direction === 'right' && direction === 'left') return
      if (this.direction === 'left' && direction === 'right') return
      if (this.direction === 'up' && direction === 'down') return
      if (this.direction === 'down' && direction === 'up') return
      let xy = this.compCoordinate(direction, this.x, this.y)

      if (this.snake.indexOf(`${xy.x}-${xy.y}`) === 1) {
        return
      }
      this.direction = direction
      // 获取方向
      // this.x = xy.x
      // this.y = xy.y

      this.changeDir = false
      setTimeout(() => {
        this.changeDir = true
      }, 30)
    },
    init () {
      let flag = 0
      // 从第几行开始
      this.x = this.random(this.snakeLen, this.rowNumber - this.snakeLen)
      // 从第几个开始
      this.y = this.random(this.snakeLen, this.columnNumber - this.snakeLen)
      // 创建🐍
      while (flag < this.snakeLen) {
        this.snake.push(this.mapLists[flag + this.x].colList[this.y].key)
        ++flag
      }
      // 创建蛋
      this.createNewEgg()
    },
    eventInit () {
      // 电脑键盘
      document.onkeydown = (e) => {
        e = e || e.event
        switch (e.keyCode) {
          case 37:
            // 向左
            this.setDirection('left')
            break
          case 38:
            // 向上
            this.setDirection('up')
            break
          case 39:
            // 向右
            this.setDirection('right')
            break
          case 40:
            // 向下
            this.setDirection('down')
            break
        }
      }
      // 手机手势
      var hammer = new Hammer(this.$refs.wrap)
      hammer.get('swipe').set({ direction: Hammer.DIRECTION_ALL })
      hammer
        .on('swipeleft', e => {
          this.setDirection('left')
        })
        .on('swiperight', e => {
          this.setDirection('right')
        })
        .on('swipeup', e => {
          this.setDirection('up')
        })
        .on('swipedown', e => {
          this.setDirection('down')
        })
    },
    // 计算行与列的数量和上下左右剩余
    computeNum () {
      this.remainTB = (window.innerHeight % 20) / 2
      this.remainLR = (window.innerWidth % 20) / 2
      this.rowNumber = (window.innerHeight - window.innerHeight % 20) / 20
      this.columnNumber = (window.innerWidth - window.innerWidth % 20) / 20
    },
    startGame () {
      return new Promise((resolve, reject) => {
        this.showCountdown = true
        var timer = setInterval(() => {
          if (this.countdown > 1) {
            --this.countdown
          } else {
            this.showCountdown = false
            clearInterval(timer)
            resolve()
          }
        }, 1000)
      })
        .then(() => {
          setTimeout(() => {
            this.countdown = 3
          }, 500)
        })
    },
    reStart () {
      location.reload()
    },
    musicPlay () {
      this.$refs.bgm.play()
      document.addEventListener('WeixinJSBridgeReady', () => {
        this.$refs.bgm.play()
      }, false)
      document.addEventListener('touchstart', () => {
        this.$refs.bgm.play()
      })
    }
  },
  watch: {
    // 分数增加，速度加快
    scoreCount (val, oldval) {
      // 每长5分就增加一次速度
      if (val > oldval && val % 5 === 0 && this.speed > 50) {
        this.speed -= 50
        clearInterval(this.moveTimer)
        this.moveTimer = setInterval(this.snakeMove, this.speed)
      }
    }
  },
  created () {
    this.computeNum()
    this.setMapData()
  },
  mounted () {
    this.musicPlay()
    this.init()
    this.startGame()
      .then(() => {
        this.eventInit()
        this.moveTimer = setInterval(this.snakeMove, this.speed)
      })
  }
}
</script>

<style scoped>
.wrap {
  margin: 0 auto;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-size: cover;
  background-image: url('../assets/images/bg.jpg');
}
.main {
  margin: 0 auto;
  height: 100%;
  position: relative;
  width: 100%;
  overflow: hidden;
}
.bgm {
  position: absolute;
  top: 999999px;
  left: 999999px;
  width: 1px;
  height: 1px;
  opacity: 0;
  font-size: 0;
  overflow: hidden;
}
.pop-ups {
  width: 100%;
  height: 100%;
  background: rgb(0, 0, 0, .5);
  position: absolute;
  top: 0;
  left: 0;
  z-index: 9;
  text-align: center;
}
.countdown-wrap__text {
  font-family: 微软雅黑;
  font-size: 200px;
  color: #fff;
  opacity: 0.8;
  height: 200px;
  line-height: 200px;
  position: absolute;
  width: 100%;
  margin: auto;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
}
.pop-ups__content {
  background: #fff;
  width: 60%;
  position: absolute;
  max-width: 320px;
  padding: 18px;
  font-family: 微软雅黑;
  top: 50%;
  left: 50%;
  transform: translate3d(-50%, -50%, 0);
}
.pop-ups__content .title {
  font-size: 22px;
  text-align: center;
  color: #333;
  font-weight: bold;
}
.pop-ups__content .desc {
  text-align: center;
  color: #666;
  font-size: 16px;
}
.code-img {
  width: 100%;
  font-size: 0;
  display: block;
}
.reStart {
  padding: 0 15px;
  height: 30px;
  line-height: 30px;
  color: #fff;
  font-size: 14px;
  background: rgb(139, 195, 74);
  box-shadow: 0;
  border: 0;
  border-radius: 15px;
  cursor: pointer;
  margin-top: 10px;
}
/* .score-col {
  width: 40%;
  height: rem(80px);
  line-height: rem(80px);
  position: absolute;
  top: 0px;
  left: 50%;
  margin-left: -20%;
  background: #000;
  text-align: center;
  border-radius: 0 0 rem(30px) rem(30px);
  opacity: .5;
  color: #ffffff;
  font-size: rem(38px);
} */
.map {
  width: 100%;
  height: 100%;
}
.map-row {
  width: 100%;
  height: 20px;
  overflow: hidden;
}
.map-col {
  height: 20px;
  width: 20px;
  float: left;
  padding: 1px;
  transition: all .3s;
}
.snake-part {
  height: 18px;
  width: 18px;
  background-size: 100% 100%;
  border-radius: 50%;
}
.snake-part.header {
  background-image: url('../assets/images/st.png');
  box-shadow: 0px 0px 1px 1px #999;
}
.snake-part.body {
  background-image: url('../assets/images/sb2.png');
  box-shadow: 0px 0px 1px 1px #999;
}
.snake-part.footer {
  background-image: url('../assets/images/sf.png');
  box-shadow: 0px 0px 1px 1px #999;
}
.snake-part.header.rotate90 {
  transform: rotate(90deg);
}
.snake-part.header.rotate180 {
  transform: rotate(180deg);
}
.snake-part.header.rotate270 {
  transform: rotate(270deg);
}
.snake-part.footer.rotate90 {
  transform: rotate(90deg);
}
.snake-part.footer.rotate180 {
  transform: rotate(180deg);
}
.snake-part.footer.rotate270 {
  transform: rotate(270deg);
}
/* .snake-part.active {
  border-radius: 35%;
  background: rgb(255, 87, 34);
} */
.snake-part.egg {
  background-image: url('../assets/images/egg.png');
}
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
.f14 {
  font-size: 14px !important;
}
.test-right {
  text-align: right !important;
}
</style>
