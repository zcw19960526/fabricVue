<template>
  <div>
    <div>
      <button @click="drawCircle">画圆</button>
      <button @click="drawLine">画线</button>
      <button @click="drawDashLine">画虚线线</button>
      <button @click="clear">清空</button>
      <button @click="selectDraw">可选中</button>
      <button @click="smile">画一个笑脸</button>
      <button @click="initCube">画一个立方体</button>
      <button @click="initYuanzhu">画一个圆柱</button>
      <button @click="initCone">画一个圆锥</button>
    </div>
    <canvas :id="id"
            :width="width"
            :height="height"></canvas>
  </div>
</template>

<script type="text/ecmascript-6">
import Utils from '../../util/index'
import down from '../../util/utils'
const dotCircleImg = require('../../assets/images/dot-circle.png')
const rotateMdrImg = require('../../assets/images/rotate-mdr.png')
import { fabric } from 'fabric'
export default {
  name: 'VueFabric',
  props: {
    id: {
      type: String,
      required: false,
      default: 'fabricCanvas'
    },
    width: {
      type: Number,
      required: true
    },
    height: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      canvas: null,
      currentObj: null,
      colors: '#000000',
      strokeWidth: 1,
      fontSize: 18,
      showColorPicker: false,
      drawingObject: null,
      textObject: null,
      mouseFrom: {},
      mouseTo: {},
      currentType: 'free',
      idDrawing: false,

      stateArr: [],
      isRedoing: false
    }
  },
  created() {},
  mounted() {
    this.canvas = new fabric.Canvas(this.id, { preserveObjectStacking: true })
    let canvas = this.canvas
    // fabric.Object.prototype.set({
    //   cornerStrokeColor: '#ff4555',
    //   cornerColor: '#ff4555',
    //   cornerStyle: 'circle',
    //   cornerSize: 28,
    //   borderScaleFactor: 6,
    //   borderColor: '#ff4555',
    //   cornerShape: 'rect', // 'rect', 'circle'
    //   cornerBackgroundColor: 'red'
    // })
    this.initEvent()
    this.initfreeDraw()
    this.setCornerIcons({})
    // canvas.add(new fabric.Circle({ radius: 30, fill: '#f55', top: 100, left: 100 }));
    canvas.backgroundColor = '#ffffff'
    // canvas.renderAll();
    // this.canvas.push(canvas);
    let that = this
    // this.canvas.controlsAboveOverlay = false
    // this.canvas.skipOffscreen = true
    // this.drawControls();
    this.canvas.on('selection:created', function (options) {
      that.$emit('selection:created', options)
    })
    this.canvas.on('mouse:down', function (options) {
      that.$emit('mouse:down', options)
    })
    this.canvas.on('mouse:up', function (options) {
      that.$emit('mouse:up', options)
    })
    this.canvas.on('mouse:move', function (options) {
      that.$emit('mouse:move', options)
    })
    this.canvas.on('mouse:dblclick', function (options) {
      that.$emit('mouse:dblclick', options)
    })
    this.canvas.on('mouse:over', function (options) {
      that.$emit('mouse:over', options)
    })
    this.canvas.on('mouse:out', function (options) {
      that.$emit('mouse:out', options)
    })
    this.canvas.on('object:added', function (options) {
      that.$emit('object:added', options)
    })
    this.canvas.on('object:removed', function (options) {
      that.$emit('object:removed', options)
    })
    this.canvas.on('object:modified', function (options) {
      that.$emit('object:modified', options)
    })
    this.canvas.on('object:rotating', function (options) {
      that.$emit('object:rotating', options)
    })
    this.canvas.on('object:scaling', function (options) {
      that.$emit('object:scaling', options)
    })
    this.canvas.on('object:moving', function (options) {
      that.$emit('object:moving', options)
    })
    this.canvas.on('selection:updated', function (options) {
      that.$emit('selection:updated', options)
    })
    this.canvas.on('selection:cleared', function (options) {
      that.$emit('selection:cleared', options)
    })
    this.canvas.on('before:selection:cleared', function (options) {
      that.$emit('before:selection:cleared', options)
    })
  },
  methods: {
    // 画圆锥
    initCone() {
      this.currentType = 'cone'
      const fabricObject = new fabric.Group(
        this.drawCone(this.mouseFrom.x, this.mouseFrom.y, this.mouseTo.x, this.mouseTo.y, {
          right: this.mouseFrom.x,
          bottom: this.mouseFrom.y,
          stroke: '#000',
          strokeWidth: 1,
          srokeLineJoin: 'round',
          fill: 'rgba(0,201,255,1)'
        })
      )
      this.toggleDrawingObject(fabricObject)
    },
    // 圆锥
    drawCone(fromX, fromY, toX, toY, pathConfig) {
      const directionX = toX - fromX
      const directionY = toY - fromY
      let sweepFlag
      if ((directionX > 0 && directionY > 0) || (directionX < 0 && directionY < 0)) {
        sweepFlag = '0'
      } else {
        sweepFlag = '1'
      }
      const huPath = 'M' + (fromX - (toX - fromX) / 2) + ',' + toY + ' A' + Math.abs(fromX - toX) / 2 + ',30,0,0,' + sweepFlag + ',' + (fromX + (toX - fromX) / 2) + ',' + toY
      const fanhuPath = 'M' + (fromX + (toX - fromX) / 2) + ',' + toY + ' A' + Math.abs(fromX - toX) / 2 + ',-30,0,0,' + sweepFlag + ',' + (fromX - (toX - fromX) / 2) + ',' + toY
      let a = { ...pathConfig, strokeDashArray: [8, 8], fill: '#ccc' }
      let b = { ...pathConfig, fill: '#ccc' }
      const hu = new fabric.Path(huPath, b)
      const banhu = new fabric.Path(fanhuPath, a)
      const c = 'M' + (fromX - (toX - fromX) / 2) + ' ' + toY + 'L' + (fromX + (toX - fromX) / 2) + ' ' + toY
      const c1 = new fabric.Path(c, Object.assign({ stroke: b.fill }), pathConfig)
      let path = 'M' + fromX + ' ' + fromY + 'L' + (fromX + (toX - fromX) / 2) + ' ' + toY
      let path1 = new fabric.Path(path, pathConfig)
      let path2 = 'M' + fromX + ' ' + fromY + 'L' + (fromX - (toX - fromX) / 2) + ' ' + toY
      let path22 = new fabric.Path(path2, pathConfig)
      const path123 = 'M' + (fromX - (toX - fromX) / 2) + ' ' + toY + 'L' + ' ' + fromX + ' ' + fromY + '' + ' ' + (fromX + (toX - fromX) / 2) + ' ' + toY + ''
      const path111 = new fabric.Path(path123, pathConfig)
      return [path111, banhu, hu, c1]
    },
    // init 圆柱
    initYuanzhu() {
      this.currentType = 'cylinder'
      const fabricObject = new fabric.Group(
        this.drawCylinder(this.mouseFrom.x, this.mouseFrom.y, this.mouseTo.x, this.mouseTo.y, {
          right: this.mouseFrom.x,
          bottom: this.mouseFrom.y,
          stroke: '#000',
          strokeWidth: 1,
          srokeLineJoin: 'round',
          fill: 'rgba(0,201,255,1)'
        })
      )
      this.toggleDrawingObject(fabricObject)
    },
    // 圆柱体
    drawCylinder(fromX, fromY, toX, toY, pathConfig) {
      const Tuoyuan = new fabric.Ellipse({
        left: fromX,
        top: fromY,
        originX: 'center',
        originY: 'center',
        rx: Math.abs(fromX - toX) / 2,
        ry: 30,
        stroke: pathConfig.stroke,
        strokeWidth: pathConfig.strokeWidth,
        fill: pathConfig.fill
      })
      const directionX = toX - fromX
      const directionY = toY - fromY
      let sweepFlag
      if ((directionX > 0 && directionY > 0) || (directionX < 0 && directionY < 0)) {
        sweepFlag = '0'
      } else {
        sweepFlag = '1'
      }
      const huPath = 'M' + (fromX - (toX - fromX) / 2) + ',' + toY + ' A' + Math.abs(fromX - toX) / 2 + ',30,0,0,' + sweepFlag + ',' + (fromX + (toX - fromX) / 2) + ',' + toY
      const fanhuPath = 'M' + (fromX + (toX - fromX) / 2) + ',' + toY + ' A' + Math.abs(fromX - toX) / 2 + ',-30,0,0,' + sweepFlag + ',' + (fromX - (toX - fromX) / 2) + ',' + toY
      const hu = new fabric.Path(huPath, pathConfig)
      let a = Object.assign({ strokeDashArray: [3, 3] }, pathConfig)
      const banhu = new fabric.Path(fanhuPath, a)
      const c = 'M' + (fromX - (toX - fromX) / 2) + ' ' + toY + 'L' + (fromX + (toX - fromX) / 2) + ' ' + toY
      const c1 = new fabric.Path(c, Object.assign({ stroke: pathConfig.fill }), pathConfig)
      const pathleft = 'M' + (fromX - (toX - fromX) / 2) + ' ' + fromY + 'L' + (fromX - (toX - fromX) / 2) + ' ' + toY
      const lineleft = new fabric.Path(pathleft, pathConfig)
      const pathright = 'M' + (fromX + (toX - fromX) / 2) + ' ' + fromY + 'L' + (fromX + (toX - fromX) / 2) + ' ' + toY + 'z'
      const lineright = new fabric.Path(pathright, pathConfig)
      return [Tuoyuan, hu, banhu, lineleft, lineright, c1]
    },
    // init cube
    initCube() {
      this.currentType = 'cube'
      const fabricObject = new fabric.Group(
        this.drawCube(this.mouseFrom.x, this.mouseFrom.y, this.mouseTo.x, this.mouseTo.y, {
          right: this.mouseFrom.x,
          bottom: this.mouseFrom.y,
          stroke: '#000',
          strokeWidth: 1,
          srokeLineJoin: 'round',
          fill: 'rgba(222,0,0,1)'
        })
      )
      this.toggleDrawingObject(fabricObject)
    },
    // 画一个立方体
    drawCube(fromX, fromY, toX, toY, pathConfig) {
      const width = toX - fromX
      const height = toY - fromY
      let path1 = 'M' + (fromX + width / 4) + ' ' + fromY
      path1 += 'L' + toX + ' ' + fromY
      path1 += 'L' + (toX - width / 4) + ' ' + (fromY + height / 4)
      path1 += 'L' + fromX + ' ' + (fromY + height / 4)
      path1 += 'z'
      let path2 = 'M' + fromX + ' ' + (fromY + height / 4)
      path2 += 'L' + fromX + ' ' + toY
      path2 += 'L' + (toX - width / 4) + ' ' + toY
      path2 += 'L' + toX + ' ' + (toY - height / 4)
      path2 += 'L' + toX + ' ' + fromY
      const path3 = 'M' + (toX - width / 4) + ' ' + (fromY + height / 4) + 'L' + (toX - width / 4) + ' ' + toY
      let path4 = 'M' + toX + ' ' + fromY
      path4 += 'L' + (toX - width / 4) + ' ' + (fromY + height / 4)
      path4 += 'L' + fromX + ' ' + (fromY + height / 4)
      return [new fabric.Path(path1, pathConfig), new fabric.Path(path2, pathConfig), new fabric.Path(path3, pathConfig), new fabric.Path(path4, pathConfig)]
    },
    // 画一个笑脸
    smile() {
      this.currentType = 'smile'
      const fabricObject = new fabric.Group(
        this.drawSmile(this.mouseFrom.x, this.mouseFrom.y, this.mouseTo.x, this.mouseTo.y, {
          right: this.mouseFrom.x,
          bottom: this.mouseFrom.y,
          stroke: 'red',
          strokeWidth: 1,
          srokeLineJoin: 'round',
          fill: 'rgba(255, 255, 255, 0)'
        })
      )
      this.toggleDrawingObject(fabricObject)
      // this.canvas.add(fabricObject)
      // this.canvas.renderAll()
    },
    // 初始化笑脸
    drawSmile(fromX, fromY, toX, toY, pathConfig) {
      const R = Math.sqrt(Math.pow(toX - fromX, 2) + Math.pow(toY - fromY, 2)) / 2
      const px = toX - fromX > 0 ? 0 : toX - fromX
      const py = toY - fromY > 0 ? 0 : toY - fromY
      const circle = new fabric.Circle({
        left: fromX + px,
        top: fromY + py,
        radius: R,
        stroke: pathConfig.stroke,
        strokeWidth: pathConfig.strokeWidth,
        fill: pathConfig.fill
      })
      const leftEye = new fabric.Circle({
        left: fromX + px + R / 2,
        top: fromY + py + R / 2,
        radius: R / 10,
        stroke: pathConfig.stroke,
        strokeWidth: pathConfig.strokeWidth,
        fill: pathConfig.stroke
      })
      const rightEye = new fabric.Circle({
        left: fromX + px + R / 3 + R,
        top: fromY + py + R / 2,
        radius: R / 10,
        stroke: pathConfig.stroke,
        strokeWidth: pathConfig.strokeWidth,
        fill: pathConfig.stroke
      })
      const mouth = new fabric.Circle({
        left: fromX + px + R / 2,
        top: fromY + py + (2 * R) / 3,
        radius: R / 2,
        angle: 0,
        startAngle: 0,
        endAngle: Math.PI,
        stroke: pathConfig.stroke,
        strokeWidth: pathConfig.strokeWidth,
        fill: pathConfig.stroke
      })
      return [circle, leftEye, rightEye, mouth]
    },
    // 绘制立方体
    lifangti(x, y, z) {
      var myCanvas = document.querySelector('#canvasId')
      var context = myCanvas.getContext('2d')
      //获取元素canvas的宽和高
      this.xwinth = myCanvas.clientWidth
      this.yheight = myCanvas.clientHeight

      this.xb = this.xwinth / 3
      this.yb = this.yheight / 3
      this.zb = Math.sqrt(Math.pow(this.xb, 2) + Math.pow(this.yb, 2)) //根据勾股定理计算出z的边长

      //x线   立方体一共12条边  x y z 分别各有四条
      context.beginPath()
      context.moveTo(this.xwinth / 3, this.yb * 2)
      context.lineTo(this.xb + (this.xwinth - this.xb) * x, this.yb * 2)

      //y线
      context.moveTo(this.xwinth / 3, this.yb * 2)
      context.lineTo(this.xwinth / 3, this.yheight - this.yb - (this.yheight - this.yb) * y)

      //z线  前提是正方形内画
      context.moveTo(this.xwinth / 3, this.yb * 2)
      //因为是正方形  所以可以得出 z在x的落点距离  和  y的落点距离
      var xluo = Math.sqrt(Math.pow(this.zb - this.zb * z, 2)) / 2
      console.log(xluo, 'x落点的距离')
      context.moveTo(xluo, this.yheight - xluo - (this.yheight - this.yb) * y)
      context.lineTo(xluo, this.yheight - xluo)
      context.lineTo(xluo + (this.xwinth - this.xb) * x, this.yheight - xluo)
      context.lineTo(xluo + (this.xwinth - this.xb) * x, this.yheight - xluo - (this.yheight - this.yb) * y)
      context.lineTo(xluo, this.yheight - xluo - (this.yheight - this.yb) * y)
      context.fillStyle = 'rgba(184, 28, 32, 0.5)'
      context.fill()
      context.lineTo(xluo + (this.xwinth - this.xb) * x, this.yheight - xluo - (this.yheight - this.yb) * y)
      context.lineTo(this.xb + (this.xwinth - this.xb) * x, this.yheight - this.yb - (this.yheight - this.yb) * y)
      context.lineTo(this.xb, this.yheight - this.yb - (this.yheight - this.yb) * y)
      context.fillStyle = 'rgba(184, 28, 32, 0.2)'
      context.fill()
      // context.globalAlpha = 0.4;
      context.lineTo(this.xb + (this.xwinth - this.xb) * x, this.yheight - this.yb - (this.yheight - this.yb) * y)
      context.lineTo(this.xb + (this.xwinth - this.xb) * x, this.yb * 2)
      context.lineTo(xluo + (this.xwinth - this.xb) * x, this.yheight - xluo)
      context.lineTo(xluo + (this.xwinth - this.xb) * x, this.yheight - xluo - (this.yheight - this.yb) * y)
      context.fillStyle = 'rgba(184, 28, 32, 0.4)'
      context.fill()
      // context.stroke()
    },
    //
    selectDraw() {
      this.removeTextObject()
      this.canvas.getObjects().map(item => {
        item.set('selectable', true)
      })
    },
    // 可选中
    removeTextObject() {
      this.currentType = ''
      if (this.textObject) {
        console.log('remove text')
        this.textObject.exitEditing()
        this.textObject = null
      }
    },
    // 清空
    clear() {
      this.canvas.clear()
      this.canvas.backgroundColor = '#efefef'
      this.resetMove()
      this.isRedoing = false
      this.stateArr = []
    },
    //
    initfreeDraw() {
      if (this.canvas == null) {
        this.canvas = new fabric.Canvas('c')
        this.canvas.backgroundColor = '#efefef'
        this.canvas.isDrawingMode = 1
        this.state = JSON.stringify(this.canvas)
      }
      this.canvas.freeDrawingBrush.color = this.colors
      this.canvas.freeDrawingBrush.width = this.width
      this.canvas.renderAll()
    },
    //
    resetMove() {
      this.mouseFrom = {}
      this.mouseTo = {}
    },
    //
    toggleDrawingObject(canvasObject) {
      this.canvas.isDrawingMode = false
      this.canvas.selection = false
      canvasObject.selectable = false
      if (this.drawingObject) {
        this.canvas.remove(this.drawingObject)
      }
      this.canvas.add(canvasObject)
      this.drawingObject = canvasObject

      if (this.textObject) {
        this.textObject.exitEditing()
        this.textObject = null
      }
    },
    //
    initCircle() {
      let left = this.mouseFrom.x
      let top = this.mouseFrom.y
      let radius = Math.sqrt((this.mouseTo.x - left) * (this.mouseTo.x - left) + (this.mouseTo.y - top) * (this.mouseTo.y - top)) / 2
      let canvasObject = new fabric.Circle({
        left: left,
        top: top,
        stroke: this.colors,
        fill: 'rgba(255, 255, 255, 0)',
        radius: radius,
        strokeWidth: this.strokeWidth
      })
      this.toggleDrawingObject(canvasObject)
    },
    // 画圆
    drawCircle() {
      this.currentType = 'circle'
      this.initCircle()
    },
    // 画虚线
    drawDashLine() {
      this.currentType = 'dash'
      this.ininDashLine()
    },
    // 画线
    drawLine() {
      this.currentType = 'line'
      this.ininLine()
    },
    // 初始化虚线
    ininDashLine() {
      let canvasObject = new fabric.Line([this.mouseFrom.x, this.mouseFrom.y, this.mouseTo.x, this.mouseTo.y], {
        fill: this.colors,
        strokeDashArray: [5, 5],
        stroke: this.colors,
        strokeWidth: this.strokeWidth
      })
      this.toggleDrawingObject(canvasObject)
    },
    // 初始化画线
    ininLine() {
      let canvasObject = new fabric.Line([this.mouseFrom.x, this.mouseFrom.y, this.mouseTo.x, this.mouseTo.y], {
        fill: this.colors,
        stroke: this.colors,
        strokeWidth: this.strokeWidth
      })
      this.toggleDrawingObject(canvasObject)
    },
    // 初始化事件
    initEvent() {
      let eventType = ['cone', 'cylinder', 'line', 'circle', 'rect', 'triangle', 'cube', 'text', 'dash', 'smile']
      this.canvas.on('mouse:down', options => {
        if (eventType.indexOf(this.currentType) != -1) {
          this.mouseFrom.x = options.pointer.x
          this.mouseFrom.y = options.pointer.y
          this.idDrawing = true
          switch (this.currentType) {
            case 'text':
              this.initText()
              break
            default:
              break
          }
        }
      })
      this.canvas.on('mouse:move', options => {
        if (this.idDrawing && eventType.indexOf(this.currentType) != -1) {
          this.mouseTo.x = options.pointer.x
          this.mouseTo.y = options.pointer.y
          switch (this.currentType) {
            case 'line':
              this.ininLine()
              break
            case 'cone':
              this.initCone()
              break
            case 'cylinder':
              this.initYuanzhu()
              break
            case 'cube':
              this.initCube()
              break
            case 'smile':
              this.smile()
              break
            case 'dash':
              this.ininDashLine()
              break
            case 'circle':
              this.initCircle()
              break
            case 'rect':
              this.initRect()
              break
            case 'triangle':
              this.initTriangle()
              break

            default:
              break
          }
        }
      })
      this.canvas.on('mouse:up', options => {
        if (eventType.indexOf(this.currentType) != -1) {
          this.mouseTo.x = options.e.clientX
          this.mouseTo.y = options.e.clientY - 50
          this.drawingObject = null
          this.idDrawing = false
          this.resetMove()
        }
      })

      this.canvas.on('object:added', () => {
        if (this.isRedoing == false) {
          this.stateArr = []
        }
      })
    },
    // 立方块儿
    cube(options) {
      const canvas = new fabric.Canvas('canvas')
      options = Object.assign({ x: 0, y: 0, x1: 0, y1: 0, color: '#B2B2B2', drawWidth: 2, fillColor: 'rgba(255,255,255,0)', theta: 35, headlen: 35 }, options)
      let canvasObject = new fabric.Path('M1,15.7L15.7,0.9h44.2v44.2L45.2,59.9H1V15.7L1,15.7L1,15.7z M1,15.7h44.2L59.9,0.9 M45.2,15.7v44.2', {
        stroke: options.color,
        fill: 'red',
        strokeWidth: '1',
        ...options
      })
      this.canvas.add(canvasObject)
      this.canvas.renderAll()
    },
    // 下载
    download() {
      const url = this.canvas.toDataURL()
      const blob = down.dataURLtoBlob(url)
      // a标签下载
      const elink = document.createElement('a')
      elink.download = '截图.png'
      elink.style.display = 'none'
      elink.href = URL.createObjectURL(blob)
      document.body.appendChild(elink)
      elink.click()
      document.body.removeChild(elink)
    },
    setCornerIcons({ size = 20, borderColor = '#e4e4e4', cornerBackgroundColor = '#ffffff', cornerShape = 'rect', tl = dotCircleImg, tr = dotCircleImg, bl = dotCircleImg, br = dotCircleImg, ml = dotCircleImg, mr = dotCircleImg, mtr = rotateMdrImg }) {
      // basic settings
      let that = this
    },
    drawDottedline(options) {
      options = Object.assign({ x: 0, y: 0, x1: 10, y1: 10, color: '#B2B2B2', drawWidth: 2, offset: 6, empty: 3 }, options)
      let canvasObject = new fabric.Line([options.x, options.y, options.x1, options.y1], {
        strokeDashArray: [options.offset, options.empty],
        stroke: options.color,
        strokeWidth: options.drawWidth,
        ...options
      })
      this.canvas.add(canvasObject)
      this.canvas.renderAll()
    },
    // 画一个箭头
    drawArrowLine(options) {
      options = Object.assign({ x: 0, y: 0, x1: 0, y1: 0, color: '#B2B2B2', drawWidth: 2, fillColor: 'rgba(255,255,255,0)', theta: 35, headlen: 35 }, options)
      let canvasObject = new fabric.Path(this.drawArrowBase(options.x, options.y, options.x1, options.y1, options.theta, options.headlen), {
        stroke: options.color,
        fill: options.fillColor,
        strokeWidth: options.drawWidth,
        ...options
      })
      this.canvas.add(canvasObject)
      this.canvas.renderAll()
    },
    drawArrowBase(fromX, fromY, toX, toY, theta, headlen) {
      theta = typeof theta !== 'undefined' ? theta : 30
      headlen = typeof theta !== 'undefined' ? headlen : 10
      // 计算各角度和对应的P2,P3坐标
      var angle = (Math.atan2(fromY - toY, fromX - toX) * 180) / Math.PI,
        angle1 = ((angle + theta) * Math.PI) / 180,
        angle2 = ((angle - theta) * Math.PI) / 180,
        topX = headlen * Math.cos(angle1),
        topY = headlen * Math.sin(angle1),
        botX = headlen * Math.cos(angle2),
        botY = headlen * Math.sin(angle2)
      var arrowX = fromX - topX,
        arrowY = fromY - topY
      var path = ' M ' + fromX + ' ' + fromY
      path += ' L ' + toX + ' ' + toY
      arrowX = toX + topX
      arrowY = toY + topY
      path += ' M ' + arrowX + ' ' + arrowY
      path += ' L ' + toX + ' ' + toY
      arrowX = toX + botX
      arrowY = toY + botY
      path += ' L ' + arrowX + ' ' + arrowY
      console.log(path)
      return path
    },
    freeDrawConfig(options) {
      options = Object.assign({ color: '#b2b2b2', drawWidth: 2 }, options)

      this.canvas.isDrawingMode = options.isDrawingMode
      this.canvas.freeDrawingBrush.color = options.color // 设置自由绘颜色
      this.canvas.freeDrawingBrush.width = options.drawWidth
      this.canvas.renderAll()
    },
    eraseDrawConfig(options) {
      options = Object.assign({ color: 'white', drawWidth: 2 }, options)

      this.canvas.freeDrawingBrush.color = options.color // 设置自由绘颜色
      this.canvas.freeDrawingBrush.width = options.drawWidth
      this.canvas.renderAll()
    },
    removeCurrentObj() {
      let obj = this.canvas.getActiveObject()
      // console.log(obj);
      this.canvas.remove(obj)
      this.canvas.renderAll()
    },
    getEditObj() {
      let obj = this.canvas.getActiveObject()
      this.removeCurrentObj()
      return obj
    },
    setEditObj(obj) {
      this.canvas.add(obj)
      this.canvas.renderAll()
    },
    // 旋转
    setRotate(deg = 36) {
      let obj = this.canvas.getActiveObject()
      let angle = obj.angle
      obj.rotate(angle + deg)
      this.canvas.renderAll()
    },
    // 提高层级
    upIt() {
      let obj = this.canvas.getActiveObject()
      this.canvas.bringForward(obj)
      this.canvas.renderAll()
    },
    discardActive() {
      this.canvas.discardActiveObject()
      // this.canvas.discardActiveGroup();
      this.canvas.renderAll()
    },
    deactivateAll() {
      // this.canvas.deactivateAll().renderAll();
    },
    deactivateOne(obj) {
      var activeGroup = this.canvas.getActiveGroup()
      activeGroup.removeWithUpdate(obj)
      this.canvas.renderAll()
    },
    setSelection(flag) {
      this.canvas.selection = flag
      this.canvas.renderAll()
    },
    moveTo() {
      let obj = this.canvas.getActiveObject()
      this.canvas.sendBackwards(obj, true)
      this.canvas.discardActiveObject()
      // this.canvas.discardActiveGroup();
    },
    createRect(options) {
      options = Object.assign({ width: 0, height: 0, fillColor: 'rgba(255, 255, 255, 0)', left: 50, top: 50 }, options)
      let rect = new fabric.Rect({
        fill: options.fillColor, // 填充的颜色
        ...options
      })
      this.canvas.add(rect)
      this.canvas.renderAll()
    },
    // 创建一个圆
    createCircle(options) {
      options = Object.assign({ left: 0, top: 0, radius: 30, fillColor: 'rgba(255, 255, 255, 0)', color: '#B2B2B2', drawWidth: 2 }, options)
      let defaultOption = {
        fill: options.fillColor,
        strokeWidth: options.drawWidth,
        stroke: options.color,
        ...options
      }
      let Circle = new fabric.Circle(defaultOption)
      this.canvas.add(Circle)
      this.canvas.renderAll()
    },
    createTriangle(options) {
      options = Object.assign({ x: 0, y: 0, x1: 0, y1: 0, x2: 0, y2: 0, left: 100, top: 100, color: '#B2B2B2', drawWidth: 2, fillColor: 'rgba(255, 255, 255, 0)', id: 'triangle' }, options)
      var path = 'M ' + options.x + ' ' + options.y + ' L ' + options.x1 + ' ' + options.y1 + ' L ' + options.x2 + ' ' + options.y2 + ' z'
      let canvasObject = new fabric.Path(path, {
        stroke: options.color,
        strokeWidth: options.drawWidth,
        fill: options.fillColor,
        ...options
      })
      this.canvas.add(canvasObject)
      this.canvas.renderAll()
    },
    createEqualTriangle(options) {
      options = Object.assign({ left: 100, top: 100, width: 50, height: 80, fillColor: 'rgba(255, 255, 255, 0)', color: '#B2B2B2', drawWidth: 2 }, options)
      // console.log(defaultOption);
      let triangle = new fabric.Triangle({
        fill: options.fillColor,
        strokeWidth: options.drawWidth,
        stroke: options.color,
        ...options
      })
      this.setContronVisibility(triangle)
      this.canvas.add(triangle)
      this.canvas.renderAll()
    },
    createLine(options) {
      options = Object.assign({ x: 0, y: 0, x1: 10, y1: 10, fillColor: 'rgba(255, 255, 255, 0)', strokeColor: '#B0B0B0' }, options)
      let line = new fabric.Line([options.x, options.y, options.x1, options.y1], {
        fill: options.fillColor,
        stroke: options.strokeColor,
        ...options
      })
      this.canvas.add(line)
      this.canvas.renderAll()
    },
    createEllipse(options) {
      options = Object.assign({ rx: 100, ry: 200, fillColor: 'rgba(255, 255, 255, 0)', angle: 90, strokeColor: '#B0B0B0', strokeWidth: 3, left: 50, top: 50 }, options)
      var ellipse = new fabric.Ellipse({
        fill: options.fillColor,
        stroke: options.strokeColor,
        ...options
      })
      this.canvas.add(ellipse)
      this.canvas.renderAll()
    },
    createText(text, options) {
      options = Object.assign({ left: 100, top: 100 }, options)
      var canvasObj = new fabric.Text(text, { ...options })
      this.canvas.add(canvasObj)
      this.canvas.renderAll()
    },
    createItext(text, options) {
      options = Object.assign({ left: 0, top: 0, fill: '#000' }, options)
      let IText = new fabric.IText(text, options)
      this.canvas.add(IText)
      this.canvas.renderAll()
    },
    createTextbox(text, options) {
      // _fontSizeMult: 5,
      options.fillColor = options.fillColor ? options.fillColor : options.fill
      options = Object.assign({ fontSize: 14, fillColor: '#000000', registeObjectEvent: false, width: 50, left: 100, top: 100 }, options)
      var canvasObj = new fabric.Textbox(text, {
        fill: options.fillColor,
        ...options
      })
      // let arr = canvasObj._splitTextIntoLines(text);
      // console.log(arr);
      this.canvas.add(canvasObj)
      if (options.registeObjectEvent) {
        Utils.registeObjectEvent(this, canvasObj)
      }
      this.canvas.renderAll()
    },
    createImageByImg(img, options) {
      options = options || {}
      let canvas = this.canvas
      let that = this
      // let maxWidth = that.width;
      let width = 0
      let height = 0
      width = img.width
      height = img.height
      // if (img.width > img.height) {
      //   if (img.width > maxWidth) {
      //     width = maxWidth;
      //     height = (img.height / img.width) * width;
      //   } else {
      //     width = img.width;
      //     height = img.height;
      //   }
      // } else {
      //   if (img.height > maxWidth) {
      //     height = maxWidth;
      //     width = (img.width / img.height) * height;
      //   } else {
      //     width = img.width;
      //     height = img.height;
      //   }
      // }
      if (options && options.width) {
        width = options.width
      }
      if (options && options.height) {
        height = options.height
      }
      let leftP = that.width / 2
      let topP = that.height / 2
      if ((options && options.left) || (options && options.left == 0)) {
        leftP = options.left + width / 2
      }
      if ((options && options.top) || (options && options.top == 0)) {
        topP = options.top + height / 2
      }
      let imgOptions = Object.assign(options, {
        id: options && options.id ? options.id : 'image',
        left: leftP,
        top: topP,
        scaleX: width / img.width,
        scaleY: height / img.height,
        originX: 'center',
        originY: 'center',
        cornerStrokeColor: 'blue'
      })
      delete imgOptions.width
      delete imgOptions.height
      var canvasImage = new fabric.Image(img, imgOptions)

      canvasImage.hasControls = true
      canvasImage.hasBorders = true

      canvas.add(canvasImage) // 把图片添加到画布上
      if (options && options.registeObjectEvent) {
        Utils.registeObjectEvent(that, canvasImage)
      }
      canvas.renderAll.bind(canvas)
    },
    createImage(url, options) {
      options = options || {}
      let canvas = this.canvas
      let that = this
      fabric.Image.fromURL(url, function (img) {
        // 添加过滤器
        // img.filters.push(new fabric.Image.filters.Grayscale());
        // 应用过滤器并重新渲染画布执行
        // img.applyFilters(canvas.renderAll.bind(canvas));
        // console.log(img);
        let maxWidth = that.width / 2
        let width = 0
        let height = 0
        width = img.width
        height = img.height
        // if (img.width > img.height) {
        //   if (img.width > maxWidth) {
        //     width = maxWidth;
        //     height = (img.height / img.width) * width;
        //   } else {
        //     width = img.width;
        //     height = img.height;
        //   }
        // } else {
        //   if (img.height > maxWidth) {
        //     height = maxWidth;
        //     width = (img.width / img.height) * height;
        //   } else {
        //     width = img.width;
        //     height = img.height;
        //   }
        // }
        if (options && options.width) {
          width = options.width
        }
        if (options && options.height) {
          height = options.height
        }
        let leftP = that.width / 2
        let topP = that.height / 2
        if ((options && options.left) || (options && options.left == 0)) {
          leftP = options.left + width / 2
        }
        if ((options && options.top) || (options && options.top == 0)) {
          topP = options.top + height / 2
        }
        // console.log(options);
        let imgOptions = Object.assign(options, {
          // ...options,
          id: options && options.id ? options.id : 'image',
          left: leftP,
          top: topP,
          scaleX: width / img.width,
          scaleY: height / img.height,
          originX: 'center',
          originY: 'center',
          cornerStrokeColor: 'blue'
        })
        delete imgOptions.width
        delete imgOptions.height
        console.log('imgOptions', imgOptions)
        img.set(imgOptions)

        var oldOriginX = img.get('originX')
        var oldOriginY = img.get('originY')
        var center = img.getCenterPoint()
        img.hasControls = true
        img.hasBorders = true
        canvas.add(img) // 把图片添加到画布上
        if (options && options.registeObjectEvent) {
          Utils.registeObjectEvent(that, img)
        }
        canvas.renderAll.bind(canvas)
      })
    },
    toJson() {
      let json = this.canvas.toJSON()
      return json
    },
    toDataUrl() {
      let canvas = this.canvas
      let dataURL = canvas.toDataURL({
        format: 'jpeg',
        quality: 1
      })
      return dataURL
    },
    loadFromJSON(json, cb) {
      let canvas = this.canvas
      canvas.loadFromJSON(json, canvas.renderAll.bind(canvas), function (o, object) {
        // `o` = json object
        // `object` = fabric.Object instance
        // ... do some stuff ...
        cb(o)
        object.setControlsVisibility({
          bl: true,
          br: true,
          mb: false,
          ml: true,
          mr: true,
          mt: false,
          mtr: true,
          tl: true,
          tr: true
        })
      })
    },
    clear() {
      this.canvas.clear()
    },
    getObjects() {
      return this.canvas.getObjects()
    },
    renderAll() {
      this.canvas.renderAll(this.canvas)
    },
    renderTop() {
      this.canvas.renderTop()
    },
    setBackgroundColor(color) {
      let canvas = this.canvas
      this.canvas.setBackgroundColor(color, canvas.renderAll.bind(canvas))
    },
    setBackgroundImage(options) {
      let canvas = this.canvas
      let opt = {
        opacity: 1,
        left: 0,
        top: 0,
        angle: 0,
        crossOrigin: null,
        originX: 'left',
        originY: 'top',
        scaleX: 1,
        scaleY: 1
      }
      // console.log(options);
      if (Object.prototype.toString.call(options) == '[object String]') {
        console.log('字符串兼容')
        opt.imgUrl = options
      } else {
        opt = Object.assign(opt, options)
      }

      // canvas.setBackgroundImage(opt.imgUrl, canvas.renderAll.bind(canvas), {
      //   opacity: opt.opacity,
      //   angle: opt.angle,
      //   left: opt.left,
      //   top: opt.top,
      //   originX: 'left',
      //   originY: 'top',
      //   crossOrigin: opt.crossOrigin,
      //   ...opt
      // });

      fabric.Image.fromURL(opt.imgUrl, function (img) {
        img.set({ width: opt.width ? opt.width : canvas.width, height: opt.height ? opt.height : canvas.height, originX: 'left', originY: 'top', scaleX: opt.scaleX, scaleY: opt.scaleY })
        canvas.setBackgroundImage(img, canvas.renderAll.bind(canvas), { ...opt })
      })
    },
    toSvg() {
      return this.canvas.toSVG()
    },
    drawControls() {
      let canvas = document.createElement('canvas')
      var ctx = canvas.getContext('2d')
      ctx.setLineDash([])
      ctx.beginPath()
      ctx.ellipse(100, 100, 50, 75, (45 * Math.PI) / 180, 0, 2 * Math.PI) // 倾斜45°角
      ctx.stroke()
      ctx.setLineDash([5])
      ctx.moveTo(0, 200)
      ctx.lineTo(200, 0)
      ctx.stroke()
      this.canvas.drawControls(ctx)
      // this.canvas.controlsAboveOverlay=true;
    },
    setContronVisibility(obj) {
      obj.setControlsVisibility({
        bl: true,
        br: true,
        mb: false,
        ml: true,
        mr: true,
        mt: false,
        mtr: true,
        tl: true,
        tr: true
      })
    },
    // 设置mirror
    toggleMirror(options) {
      options = options || {}
      options = Object.assign({ flip: 'X' }, options)
      let img = this.canvas.getActiveObject()
      // if (img && img.type == 'image') {
      if (options.flip === 'X') {
        img.toggle('flipX')
      } else {
        img.toggle('flipY')
      }
      this.renderAll()
      // }
    },
    // 设置层级
    toNextLayer() {
      let obj = this.canvas.getActiveObject()
      if (!obj) {
        return
      }
      obj.sendBackwards(true)
      this.renderTop()
      // this.canvas.setActiveObject(obj);
    },
    toBottomLayer() {
      let obj = this.canvas.getActiveObject()
      if (!obj) {
        return
      }
      obj.sendToBack()
      this.renderTop()
      // this.canvas.setActiveObject(obj);
    },
    toLastLayer() {
      let obj = this.canvas.getActiveObject()
      if (!obj) {
        return
      }
      obj.bringForward(true)
      this.renderTop()
    },
    toTopLayer() {
      let obj = this.canvas.getActiveObject()
      if (!obj) {
        return
      }
      obj.bringToFront()
      this.renderTop()
    },
    drawByPath(pathArray, options) {
      options = Object.assign({ fillColor: 'rgba(255, 255, 255, 0)', left: 150, top: 150, strokeColor: '#B0B0B0', strokeWidth: 3 }, options)
      let pathStr = 'M '
      for (let item of pathArray) {
        pathStr = pathStr + item[0] + ' ' + item[1] + ' '
      }
      pathStr = pathStr + 'z'
      var path = new fabric.Path(pathStr)
      path.set({
        stroke: options.strokeColor,
        fill: options.fillColor,
        ...options
      })
      this.canvas.add(path)
    }
  }
}
</script>

<style lang="less" scoped></style>
