<template>
  <div class="main">
    <div class="select-main">
      <button class="button" v-for="(item, index) in selectData" :key="index" :class="{active: activeIndex == index ? true : false}" @click="btnClick(index)">
        {{ item }}
      </button>
    </div>
    <div class="canvas-main">
      <canvas id="pic" width="1800" height="600">
      </canvas>
      <div class="coordinates">
        x: {{ x }}, y: {{ y }}
      </div>
    </div>
  </div>
</template>

<script>
import { fabric } from "fabric"

export default {
  data() {
    return {
      selectData: ['Hand', 'Bezier Curve', 'Line', 'Detect', 'Delete'],
      activeIndex: null,
      x: "",
      y: "",
      canvasModel: "",
      id: 0,
      followPoint: null,
      followLine: null,
      // 是否是绘制状态
      lineStatus: false,
      // 是否执行过up
      upStatus: false,
      downPoint: null,
      curve: null,
      // 是否正在绘制曲线
      ongoing: false,
      pointList: []
    }
  },
  mounted() {
    this.fabricInit()
  },
  watch: {
    activeIndex(value) {
      if(value == 1) {
        this.canvasModel.on({
          'mouse:move:before': this.onObjectMoveBrfore,
          'mouse:move': this.onObjectMoveCurve,
          'mouse:down': this.onObjectDown,
          'mouse:up': this.onObjectUp,
          'mouse:out': this.onObjectOut,
        });
      } else if (value == 2) {
        this.canvasModel.on({
          'mouse:move:before': this.onObjectMoveBrfore,
          'mouse:move': this.onObjectMoveLine,
          // 'mouse:down': this.onObjectDown,
          'mouse:up': this.onObjectUp,
          'mouse:out': this.onObjectOut,
        });
      }
    }
  },
  methods: {
    btnClick(value) {
      this.activeIndex = value
    },
    fabricInit() {
      this.canvasModel = new fabric.Canvas('pic');
      fabric.Object.prototype.originX = fabric.Object.prototype.originY = 'center';
      this.followPoint = new fabric.Circle({
        left: 1,
        top: 1,
        strokeWidth: 1,
        radius: 4,
        fill: '#fff',
        stroke: '#666'
      })
      this.followLine = new fabric.Line([0, 0, 0, 0], {
        left: 0,
        top: 0,
        stroke: 'black'
      })
      // this.curve = new fabric.Path(`M 0 0 C 0 0 0 0 0 0`)
      // this.curve.set({
      //   fill: 'black',
      //   stroke: 'red'
      // })
      this.canvasModel.add(this.followPoint)
      this.canvasModel.add(this.followLine)
      // this.canvasModel.add(this.curve)
      // this.canvasModel.on({
      //   'mouse:move:before': this.onObjectMoveBrfore,
      //   'mouse:move': this.onObjectMove,
      //   'mouse:down': this.onObjectDown,
      //   'mouse:up': this.onObjectUp,
      //   'mouse:out': this.onObjectOut,
      // });
      let rect = new fabric.Rect({
    
          left:100,//距离画布左侧的距离，单位是像素
    
          top:100,//距离画布上边的距离
    
          fill:'red',//填充的颜色
    
          width:30,//方形的宽度
    
          height:30//方形的高度
      })
      this.canvasModel.add(rect)
      let path = new fabric.Path('M 65 0 C 100, 100, 200, 0')
      path.set({
        left: 120,
        top: 120,
        fill: '',
        stroke: 'black'
      })
      path.path[0][1] = 100;
      path.path[0][2] = 100;

      path.path[1][1] = 200;
      path.path[1][2] = 200;

      path.path[1][3] = 300;
      path.path[1][4] = 100;
      this.canvasModel.add(path)
      // console.log(canvas)
    },
    onObjectMoveBrfore() {
    },
    onObjectMoveLine(e) {
      console.log(this.curve)
      this.x = e.pointer.x
      this.y = e.pointer.y
      this.followPoint.set({
        left: e.pointer.x,
        top: e.pointer.y
      });
      if(this.lineStatus) {
        this.followLine.set({
          x1: e.pointer.x,
          y1: e.pointer.y
        })
      }
      // this.followPoint.setCoords();
      this.canvasModel.renderAll()
    },
    onObjectMoveCurve(e) {
      // console.log(this.curve)
      this.x = e.pointer.x
      this.y = e.pointer.y
      this.followPoint.set({
        left: e.pointer.x,
        top: e.pointer.y
      });
      if(this.lineStatus && !this.upStatus) {
        const prev = this.pointList[this.pointList.length - 2]
        const current = this.pointList[this.pointList.length - 1]
        const controlX = current.left - (this.x - current.left)
        const controlY = current.top - (this.y - current.top)
        console.log(prev, current)
        if(this.pointList.length >= 2) {
          console.log(prev.left, prev.top, controlX, controlY, current.left, current.top)
          this.canvasModel.remove(this.curve)
          this.curve = new fabric.Path(`M ${prev.left} ${prev.top} C ${prev.left} ${prev.top} ${controlX} ${controlY} ${current.left} ${current.top}`)
          this.curve.set({
            fill: '',
            stroke: 'black'
          })
          this.canvasModel.add(this.curve)
        } else {
          return
        }
      } else if(this.lineStatus) {
        // const prev = this.pointList[this.pointList.length - 2]
        const current = this.pointList[this.pointList.length - 1]
        // const controlX = current.left - (this.x - current.left)
        // const controlY = current.top - (this.y - current.top)
        // console.log(2);
        this.ongoing = true
        this.canvasModel.remove(this.curve)
        this.curve = new fabric.Path(`M ${current.left} ${current.top} C ${current.left} ${current.top} ${this.x} ${this.y} ${this.x} ${this.y}`)
        this.curve.set({
          fill: '',
          stroke: 'black'
        })
        this.canvasModel.add(this.curve)
        // const path = ['C', this.downPoint.left, this.downPoint.top, this.x, this.y, this.x, this.y]
        // this.curve.path[1] = path
        // this.curve.path[1][3] = this.x
        // this.curve.path[1][4] = this.y
        // this.curve.path[1][5] = this.x
        // this.curve.path[1][6] = this.y
        // this.curve.setCoords()
        // console.log(this.curve);
      }
      this.canvasModel.renderAll()
      // this.followPoint.setCoords();
    },
    onObjectDown(e) {
      const p = this.makeCirclePoint(e.pointer.x, e.pointer.y)
      this.upStatus = false
      if(this.activeIndex == 1) {
        if(this.lineStatus) {
          this.downPoint = p
          // this.downPoint = this.deepClone(p)
          this.pointList.push(this.downPoint)
          console.log(this.downPoint, '后');
          this.canvasModel.add(this.downPoint)
          // this.downPoint = null
          // this.lineStatus = false
          // this.id++
          // console.log(this.curve)
        } else {
          this.downPoint = p
          // this.downPoint = this.deepClone(p)
          this.pointList.push(this.downPoint)
          console.log(this.downPoint, '前');
          if(!this.origin) {
            this.origin = p
          }
          // console.log(e)
          this.lineStatus = true
          // this.curve = new fabric.Path(`M ${this.x} ${this.y} C ${this.x} ${this.y} ${this.x} ${this.y} ${this.x} ${this.y}`)
          // this.curve.set({
          //   stroke: 'black',
          //   fill: 'red'
          // })
          // this.canvasModel.add(this.curve)
          this.canvasModel.add(this.downPoint)
          // this.curve.path[0][1] = this.x
          // this.curve.path[0][2] = this.y
          // this.curve.path[1][1] = this.x
          // this.curve.path[1][2] = this.y
          // this.curve.path[1][3] = this.x
          // this.curve.path[1][4] = this.y
          // this.curve.path[1][5] = this.x
          // this.curve.path[1][6] = this.y
        }
      }else if(this.activeIndex == 2) {
        if(this.lineStatus) {
          const line = this.followLine
          const L = new fabric.Line([line.x1, line.y1, line.x2, line.y2], {
            stroke: 'black',
            id: this.id
          })
          this.canvasModel.add(L)
          this.followLine.set({
            x1: 0,
            y1: 0,
            x2: 0,
            y2: 0
          })
          this.canvasModel.remove(this.downPoint)
          this.downPoint = null
          this.lineStatus = false
          this.id++
        } else {
          this.downPoint = p
          console.log(e)
          this.lineStatus = true
          this.canvasModel.add(this.downPoint)
          this.followLine.set({
            x1: this.x,
            y1: this.y,
            x2: this.x,
            y2: this.y
          })
        }
      }
      this.canvasModel.renderAll()
    },
    onObjectUp() {
      this.upStatus = true
      if(this.pointList.length >= 2) {
        console.log(this.pointList);
        const val = this.curve
        const curve = new fabric.Path(`M ${val.path[0][1]} ${val.path[0][2]} C ${val.path[1][1]} ${val.path[1][2]} ${val.path[1][3]} ${val.path[1][4]} ${val.path[1][5]} ${val.path[1][6]}`)
        curve.set({
          fill: '',
          stroke: 'black'
        })
        console.log(curve)
        this.canvasModel.add(curve)
      }
      // if(this.ongoing) {
      //   //
      // } else {
      //   //
      // }
    },
    makeCirclePoint(left, top) {
      const c = new fabric.Circle({
        left,
        top,
        strokeWidth: 1,
        radius: 4,
        fill: '#fff',
        stroke: '#666'
      })
      return c
    },
    onObjectOut() {

    },
    createBezier() {

    },
    // 深拷贝
    deepClone(obj) {
      var _tmp,result;
      _tmp = JSON.stringify(obj);
      result = JSON.parse(_tmp);
      return result;
    },
    cancelDefault(e) {
      e.preventDefault()
      e.stopPropagation()
      return false
    }
  }
}
</script>

<style>
.main {
  width: 100%;
  height: 100%;
  padding: 30px;
  box-sizing: border-box;
  position: relative;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-content: flex-start;
}
.select-main {
  width: 100%;
  height: 100px;
  display: flex;
  justify-content: flex-start;
  align-items: center;
}
.button {
  box-sizing: border-box;
  border: 1px solid grey;
  border-radius: 0.2rem;
  padding: 10px 10px;
  margin-right: 12px;
}
.active {
  border-color:blue;
  color: blue;
}
.canvas-main {
  width: 100%;
  height: 600px;
  border: 1px solid grey;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
}
.coordinates {
  height: 50px;
  display: flex;
  align-items: center;
}
.canvas-main>#pic {
  width: 100%;
  box-sizing: border-box;
  background-color: #F5F5F5;
}
</style>