<template>
  <div class="canvas_container">
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script>
export default {
  name: 'canvasBg',
  data () {
    return {
      stars: {
        dotDensity: 3, // 星点的密集度n（每1w像素中有n个点）
        lineDistance: 80, // 连线间的最大距离
        mouseDistance: 60, // 鼠标交互距离
        dotSize: [0.2, 1.2], // 星点尺寸范围
        lineSize: 0.8, // 线条粗细
        dotSpeed: 0.2, // 星点运动速度,
        bgColor: '#000', // 背景色
        dotColor: '#F03252', // 点的颜色
        lineColor: '255, 255, 255', // 连线颜色
        dotCount: undefined
      },
      screenIsChanged: true,
      width: undefined,
      height: undefined,
      ctx: undefined,
      points: undefined,
      cursorX: undefined,
      cursorY: undefined
    }
  },
  mounted () {
    // 启动动画
    this.start()
    // 监听屏幕变化初始化数据
    let self = this
    window.onresize = function () {
      self.screenIsChanged = true
    }
  },
  methods: {
    // 初始化函数init()
    init () {
      this.stars.dotCount = Math.ceil((innerHeight * innerWidth) / 10000 * this.stars.dotDensity)
      let canvas = this.$refs.canvas
      canvas.width = this.width = innerWidth
      canvas.height = this.height = innerHeight
      canvas.style.background = this.stars.bgColor
      this.ctx = canvas.getContext('2d')
      this.points = new Array(this.stars.dotCount)
      // 实例化数组中的每个星点
      for (let i = 0; i < this.stars.dotCount; i++) {
        this.points[i] = new this.Dot(this, this.width, this.height, this.stars.dotSpeed, this.stars.dotSize, this.ctx, this.stars.dotColor)
      }
    },
    // 生成范围内的随机数
    getRandom (min, max) {
      return Math.random() * (max - min) + min
    },
    // 计算两坐标间距
    calculateDistance (x1, y1, x2, y2) {
      return Math.sqrt((x1 - x2) * (x1 - x2) + (y1 - y2) * (y1 - y2))
    },
    // 绘制鼠标范围内的两点间的线条
    drawLine (ctx, x1, y1, x2, y2, cursorX, cursorY) {
      ctx.beginPath()
      ctx.moveTo(x1, y1)
      ctx.lineTo(x2, y2)
      ctx.lineWidth = this.stars.lineSize
      // 定义透明值
      let opacity = 0
      // 计算线的中点坐标
      let midX = (x1 + x2) / 2
      let midY = (y1 + y2) / 2
      // 根据鼠标与中点间的距离是否在范围内设置线条渐变透明度
      if (this.calculateDistance(midX, midY, cursorX, cursorY) < this.stars.mouseDistance) {
        opacity = ((1 - this.calculateDistance(midX, midY, cursorX, cursorY) / this.stars.mouseDistance) * 0.8).toFixed(2)
      }
      ctx.strokeStyle = `rgba(${this.stars.lineColor}, ${opacity})`
      ctx.stroke()
    },
    // 随机星点构造函数
    Dot (self, w, h, s, range, ctx, color) {
      this.dotX = self.getRandom(1, w)
      this.dotY = self.getRandom(1, h)
      this.dotVX = self.getRandom(-s, s)
      this.dotVY = self.getRandom(-s, s)
      this.dotR = self.getRandom(range[0], range[1])
      // 更新点的位置
      this.moveDot = function () {
        this.dotX += this.dotVX
        this.dotY += this.dotVY
        // 检测边缘碰撞
        if (this.dotX + this.dotVX > w || this.dotX + this.dotVX < 0) {
          this.dotVX = -this.dotVX
        }
        if (this.dotY + this.dotVY > h || this.dotY + this.dotVY < 0) {
          this.dotVY = -this.dotVY
        }
      }
      // 绘制星点
      this.drawDot = function () {
        ctx.beginPath()
        ctx.arc(this.dotX, this.dotY, this.dotR, 0, Math.PI * 2, true)
        ctx.closePath()
        ctx.fillStyle = color
        ctx.fill()
      }
    },
    // 周期函数draw()
    draw () {
      // 初始化
      let self = this
      if (this.screenIsChanged) {
        this.init()
        this.screenIsChanged = false
      }
      // 获取鼠标坐标
      document.onmousemove = function (e) {
        self.cursorX = e.pageX
        self.cursorY = e.pageY
      }
      // 清空画布
      this.ctx.clearRect(0, 0, this.width, this.height)
      // 移动点
      for (let i = 0; i < this.stars.dotCount; i++) {
        this.points[i].moveDot()
      }
      // 根据条件画线
      for (let i = 0; i < this.stars.dotCount; i++) {
        for (let j = i + 1; j < this.stars.dotCount; j++) {
          if (this.calculateDistance(this.points[i].dotX, this.points[i].dotY, this.points[j].dotX, this.points[j].dotY) < this.stars.lineDistance) {
            this.drawLine(this.ctx, this.points[i].dotX, this.points[i].dotY, this.points[j].dotX, this.points[j].dotY, this.cursorX, this.cursorY)
          }
        }
      }
      // 在画线之后画点
      for (let i = 0; i < this.stars.dotCount; i++) {
        this.points[i].drawDot()
      }
      window.requestAnimationFrame(this.draw)
    },
    // 动画设置
    start () {
      // 执行循环动画
      window.requestAnimationFrame(this.draw)
    }
  }
}
</script>

<style scoped>
  .canvas_container {
    position: absolute;
    overflow: hidden;
    z-index: -1;
    width: 100%;
    height: 100%;
  }
</style>
