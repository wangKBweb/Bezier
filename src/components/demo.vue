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
      pointList: [],
      symmetryX: null,
      symmetryY: null,
      startX: null,
      startY: null,
      controlX: null,
      controlY: null,
      controlX2: null,
      controlY2: null,
      endX: null,
      endY: null,
      ongoing2: false,
      nodeList: [],
      lineList: [],
      // 是否在原点
      originStatus: false
    }
  },
  mounted() {
    this.fabricInit()
    window.addEventListener("keydown", (e) => {
      if(e.keyCode == 27) {
        this.drawEnd()
      }
    })
  },
  watch: {
    activeIndex(value) {
      if(this.activeIndex !== null) {
        this.eventClear()
        this.dataInit()
      }
      if(value == 0) {
        this.canvasModel.on({
          'mousedblclick': this.onObjectDblclick
        });
      } else if (value == 1) {
        this.canvasModel.on({
          'mouse:move': this.onObjectMoveCurve,
          'mouse:down': this.onObjectDown,
          'mouse:up': this.onObjectUp,
          'mouse:out': this.onObjectOut,
          'mouse:over': this.onObjectOver
        });
      } else if (value == 2) {
        this.canvasModel.on({
          'mouse:move': this.onObjectMoveLine,
          'mouse:down': this.onObjectDown,
          'mouse:up': this.onObjectUp,
          'mouse:out': this.onObjectOut,
          'mouse:over': this.onObjectOver
        });
      } else if (value == 3) {
        this.canvasModel.on({
          'mouse:move': this.onObjectMoveDetect,
          'mouse:down': this.onObjectDown,
          'mouse:up': this.onObjectUp,
          'mouse:out': this.onObjectOut,
          'mouse:over': this.onObjectOver
        });
      }
    }
  },
  methods: {
    btnClick(value) {
      this.activeIndex = value
    },
    eventClear() {
      this.canvasModel.__eventListeners['mouse:move'] = []
      this.canvasModel.__eventListeners['mouse:down'] = []
      this.canvasModel.__eventListeners['mouse:up'] = []
      this.canvasModel.__eventListeners['mouse:out'] = []
      this.canvasModel.__eventListeners['mouse:move'] = []
    },
    fabricInit() {
      this.canvasModel = new fabric.Canvas('pic');
      fabric.Object.prototype.originX = fabric.Object.prototype.originY = 'center';
      fabric.Object.prototype.hoverCursor = 'default'
      fabric.Object.prototype.moveCursor = 'default'
      console.log(fabric.Object.prototype, 'Object')
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
      this.canvasModel.on({
        'mouse:move:before': this.onObjectMoveBrfore,
        'mouse:move': this.onObjectMove,
        'mouse:down': this.onObjectDown,
        'mouse:up': this.onObjectUp,
        'mouse:out': this.onObjectOut,
        'mousedblclick': this.onObjectDblclick
      });
      // let rect = new fabric.Rect({
    
      //     left:100,//距离画布左侧的距离，单位是像素
    
      //     top:100,//距离画布上边的距离
    
      //     fill:'red',//填充的颜色
    
      //     width:30,//方形的宽度
    
      //     height:30//方形的高度
      // })
      // this.canvasModel.add(rect)
      // let path = new fabric.Path('M 65 0 C 100, 100, 200, 0')
      // path.set({
      //   left: 120,
      //   top: 120,
      //   fill: '',
      //   stroke: 'black'
      // })
      // path.path[0][1] = 100;
      // path.path[0][2] = 100;

      // path.path[1][1] = 200;
      // path.path[1][2] = 200;

      // path.path[1][3] = 300;
      // path.path[1][4] = 100;
      // this.canvasModel.add(path)
    },
    onObjectMoveBrfore() {
    },
    onObjectMoveLine(e) {
      console.log(111);
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
      this.canvasModel.renderAll()
    },
    onObjectMoveCurve(e) {
      this.x = e.pointer.x
      this.y = e.pointer.y
      // const prev = this.pointList[this.pointList?.length - 2]
      const current = this.pointList[this.pointList?.length - 1]
      const controlX = current ? current.left - (this.x - current.left) : null
      const controlY = current ? current.top - (this.y - current.top) : null
      this.followPoint.set({
        left: e.pointer.x,
        top: e.pointer.y
      });
      // if(this.ongoing) {
      //   this.canvasModel.remove(this.curve)
      //   this.curve = new fabric.Path(`M ${current.left} ${current.top} C ${this.symmetryX} ${this.symmetryY} ${this.followPoint.left} ${this.followPoint.top} ${this.followPoint.left} ${this.followPoint.top}`)
      //   this.curve.set({
      //     fill: '',
      //     stroke: 'black'
      //   })
      //   this.canvasModel.add(this.curve)
      // }
      if(this.ongoing) {
        this.controlX2 = controlX
        this.controlY2 = controlY
        this.endX = current?.left
        this.endY = current?.top
        if(this.ongoing2) {
          this.controlX2 = this.x
          this.controlY2 = this.y
          this.endX = this.x
          this.endY = this.y
        }
      } else {
        this.controlX2 = this.x
        this.controlY2 = this.y
        this.endX = this.x
        this.endY = this.y
      }
      if(this.lineStatus && !this.upStatus) {
        if(this.pointList.length >= 2) {
          // console.log(prev.left, prev.top, controlX, controlY, current.left, current.top)
          this.canvasModel.remove(this.curve)
          // 原点计算处理
          this.originHandle()
          this.curve = new fabric.Path(`M ${this.startX} ${this.startY} C ${this.controlX} ${this.controlY} ${this.controlX2} ${this.controlY2} ${this.endX} ${this.endY}`)
          console.log(this.startX, this.startY, this.controlX, this.controlY, this.controlX2, this.controlY2, this.endX, this.endY);
          this.curve.set({
            fill: '',
            stroke: 'black'
          })
          this.canvasModel.add(this.curve)
          this.ongoing = true
        } else {
          return
        }
      } else if(this.lineStatus) {
        this.canvasModel.remove(this.curve)
        // 原点计算处理
        this.originHandle()
        this.curve = new fabric.Path(`M ${current.left} ${current.top} C ${current.left} ${current.top} ${this.controlX2} ${this.controlY2} ${this.endX} ${this.endY}`)
        this.curve.set({
          fill: '',
          stroke: 'black'
        })
        this.canvasModel.add(this.curve)
        this.ongoing = false
      }
      this.canvasModel.renderAll()
      // this.followPoint.setCoords();
    },
    onObjectMoveDetect(e) {
      console.log(e);
      console.log(this.canvasModel)
    },
    onObjectDown(e) {
      const p = this.makeCirclePoint(e.pointer.x, e.pointer.y)
      this.upStatus = false
      console.log(this.origin);
      if(this.activeIndex == 1) {
        if(this.lineStatus) {
          this.downPoint = p
          // this.downPoint = this.deepClone(p)
          this.pointList.push(this.downPoint)
          this.canvasModel.add(this.downPoint)
          // this.downPoint = null
          // this.lineStatus = false
          if(this.ongoing && this.ongoing2) {
            this.ongoing = false
            this.ongoing2 = false
          }
          if(this.originStatus) {
            this.nodeList.push({
              control1: [this.curve.path[1][1], this.curve.path[1][2]],
              control2: [this.curve.path[1][3], this.curve.path[1][4]],
              end: [this.curve.path[1][5], this.curve.path[1][6]]
            })
            this.drawEnd()
            return
          }
          // this.id++
          // console.log(this.curve)
        } else {
          console.log('true');
          this.downPoint = p
          // this.downPoint = this.deepClone(p)
          this.pointList.push(this.downPoint)
          this.nodeList.push({
            start: [e.pointer.x, e.pointer.y]
          })
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
      const val = this.curve
      // const prev = this.pointList[this.pointList?.length - 1]
      // const controlX = current ? current.left - (this.x - current.left) : null
      // const controlY = current ? current.top - (this.y - current.top) : null
      if(this.pointList.length >= 2) {
        console.log(this.pointList);
        this.symmetryX = this.x
        this.symmetryY = this.y
        const curve = new fabric.Path(`M ${val.path[0][1]} ${val.path[0][2]} C ${val.path[1][1]} ${val.path[1][2]} ${val.path[1][3]} ${val.path[1][4]} ${val.path[1][5]} ${val.path[1][6]}`)
        curve.set({
          fill: '',
          stroke: 'black'
        })
        this.canvasModel.add(curve)
        this.lineList.push(curve)
        this.nodeList.push({
          // start: [curve.path[0][1], curve.path[0][2]],
          control1: [curve.path[1][1], curve.path[1][2]],
          control2: [curve.path[1][3], curve.path[1][4]],
          end: [curve.path[1][5], curve.path[1][6]]
        })
      }
      if(this.ongoing) {
        console.log(true);
        this.startX = val.path[1][5]
        this.startY = val.path[1][6]
        this.controlX = this.x
        this.controlY = this.y
        this.upStatus = false
        this.ongoing2 = true
        //
      } else {
        console.log(false);
        const prev = this.pointList[this.pointList?.length - 1]
        if(prev) {
          this.startX = prev.left
          this.startY = prev.top
          this.controlX = prev.left
          this.controlY = prev.top
        }
        this.upStatus = true
        //
      }
    },
    originHandle() {
      const x = this.origin ? this.x - this.origin.left : 100
      const y = this.origin ? this.y - this.origin.top : 100
      if(x <= 6 && x >= -6 && y <= 6 && y >= -6) {
        console.log(1212);
        this.followPoint.set({
          left: this.origin.left,
          top: this.origin.top
        })
        this.endX = this.origin.left
        this.endY = this.origin.top
        this.controlX2 = this.origin.left
        this.controlY2 = this.origin.top
        this.originStatus = true
      } else {
        this.originStatus = false
      }
    },
    drawEnd() {
      if(this.activeIndex == 1) {
        const node = this.nodeList.slice(1)
        const line = []
        line.push(`M ${this.nodeList[0].start[0]} ${this.nodeList[0].start[1]} `)
        node.forEach(item => {
          const str = `C ${item.control1[0]} ${item.control1[1]} ${item.control2[0]} ${item.control2[1]} ${item.end[0]} ${item.end[1]} `
          line.push(str)
        })
        console.log(line.join(""))
        this.lineList.forEach(item => {
          this.canvasModel.remove(item)
        })
        this.pointList.forEach(item => {
          this.canvasModel.remove(item)
        })
        const curve = new fabric.Path(line.join(""))
        curve.set({
          fill: '',
          stroke: 'black'
        })
        this.canvasModel.add(curve)
        this.dataInit()
      } else if(this.activeIndex == 2) {
        //
      }
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
    onObjectDblclick(e) {
      console.log(e, '我是双击');
    },
    onObjectOver(e) {
      if (e.target != null) {
        e.target.hoverCursor = this.canvasModel.defaultCursor;
      } 
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
    // 重置参数
    dataInit() {
      this.pointList = []
      this.lineList = []
      this.nodeList = []
      this.lineStatus = false
      this.ongoing = false
      this.ongoing2 = false
      this.upStatus = false
      this.originStatus = false
      this.origin = null
      this.followLine.set({
        left: 0,
        top: 0
      })
      this.canvasModel.remove(this.curve)
      // this.curve
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