<template>
  <div v-bind="{ scopedSlots: $scopedSlots }"
    class="orgchart-container"
    @wheel="zoom && zoomHandler($event)"
    @mouseup="pan && panning && panEndHandler($event)"
  >
    <div
      class="orgchart"
      :style="{ transform: transformVal, cursor: cursorVal }"
      @mousedown="pan && panStartHandler($event)"
      @mousemove="pan && panning && panHandler($event)"
    >
      <organization-chart-node :datasource="datasource" :handle-click="handleClick" :active-node="activeNode">
        <template v-for="slot in Object.keys($scopedSlots)" :slot="slot" slot-scope="scope">
          <slot :name="slot" v-bind="scope"/>
        </template>
      </organization-chart-node>
    </div>
  </div>
</template>

<script>
import $ from 'jquery'
import OrganizationChartNode from './OrganizationChartNode.vue'

export default {
  name: 'Container',
  props: {
    datasource: {
      type: Object,
      required: true
    },
    activeNode: {
      type: Object,
      required: false
    },
    pan: {
      type: Boolean,
      required: false
    },
    zoom: {
      type: Boolean,
      required: false
    },
    zoomoutLimit: {
      type: Number,
      required: false,
      default: 0.5
    },
    zoominLimit: {
      type: Number,
      required: false,
      default: 7
    }
  },
  data () {
    return {
      cursorVal: 'default',
      panning: false,
      startX: 0,
      startY: 0,
      transformVal: ''
    }
  },
  components: {
    OrganizationChartNode
  },
  methods: {
    handleClick (nodeData) {
      this.$emit('node-click', nodeData);
    },
    panEndHandler () {
      this.panning = false
      this.cursorVal = 'default'
    },
    panHandler (e) {
        let newX = 0
        let newY = 0
        if (!e.targetTouches) { // pand on desktop
          newX = e.pageX - this.startX
          newY = e.pageY - this.startY
        } else if (e.targetTouches.length === 1) { // pan on mobile device
          newX = e.targetTouches[0].pageX - this.startX
          newY = e.targetTouches[0].pageY - this.startY
        } else if (e.targetTouches.length > 1) {
          return;
        }
        if (this.transformVal === '') {
          if (this.transformVal.indexOf('3d') === -1) {
            this.transformVal = 'matrix(1,0,0,1,' + newX + ',' + newY + ')'
          } else {
            this.transformVal = 'matrix3d(1,0,0,0,0,1,0,0,0,0,1,0,' + newX + ', ' + newY + ',0,1)'
          }
        } else {
          let matrix = this.transformVal.split(',')
          if (this.transformVal.indexOf('3d') === -1) {
            matrix[4] = newX
            matrix[5] = newY + ')'
          } else {
            matrix[12] = newX
            matrix[13] = newY
          }
          this.transformVal = matrix.join(',')
        }
    },
    panStartHandler (e) {
      if ($(e.target).closest('.node').length) {
        this.panning = false
        return
      } else {
        this.cursorVal = 'move'
        this.panning = true
      }
      let lastX = 0
      let lastY = 0
      if (this.transformVal !== '') {
        let matrix = this.transformVal.split(',')
        if (this.transformVal.indexOf('3d') === -1) {
          lastX = parseInt(matrix[4])
          lastY = parseInt(matrix[5])
        } else {
          lastX = parseInt(matrix[12])
          lastY = parseInt(matrix[13])
        }
      }
      if (!e.targetTouches) { // pand on desktop
        this.startX = e.pageX - lastX
        this.startY = e.pageY - lastY
      } else if (e.targetTouches.length === 1) { // pan on mobile device
        this.startX = e.targetTouches[0].pageX - lastX
        this.startY = e.targetTouches[0].pageY - lastY
      } else if (e.targetTouches.length > 1) {
        return
      }
    },
    setChartScale (newScale) {
      let matrix = ''
      let targetScale = 1
      if (this.transformVal === '') {
        this.transformVal = 'matrix(' + newScale + ', 0, 0, ' + newScale + ', 0, 0)'
      } else {
        matrix = this.transformVal.split(',')
        if (this.transformVal.indexOf('3d') === -1) {
          targetScale = Math.abs(window.parseFloat(matrix[3]) * newScale)
          if (targetScale > this.zoomoutLimit && targetScale < this.zoominLimit) {
            matrix[0] = 'matrix(' + targetScale
            matrix[3] = targetScale
            this.transformVal = matrix.join(',')
          }
        } else {
          targetScale = Math.abs(window.parseFloat(matrix[5]) * newScale)
          if (targetScale > this.zoomoutLimit && targetScale < this.zoominLimit) {
            matrix[0] = 'matrix3d(' + targetScale
            matrix[5] = targetScale
            this.transformVal = matrix.join(',')
          }
        }
      }
    },
    zoomHandler (e) {
      let newScale  = 1 + (e.deltaY > 0 ? -0.2 : 0.2)
      this.setChartScale(newScale)
    }
  }
};
</script>

<style>
.orgchart-container {
  position: relative;
  display: inline-block;
  height: 420px;
  width: calc(100% - 24px);
  border: 2px dashed #aaa;
  border-radius: 5px;
  overflow: auto;
  text-align: center;
}
.orgchart {
  box-sizing: border-box;
  display: inline-block;
  min-height: 202px;
  min-width: 202px;
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  background-image: linear-gradient(
      90deg,
      rgba(80, 79, 79, 0.15) 10%,
      rgba(0, 0, 0, 0) 10%
    ),
    linear-gradient(rgba(116, 114, 114, 0.15) 10%, rgba(0, 0, 0, 0) 10%);
  background-size: 10px 10px;
  border: 1px dashed rgba(0, 0, 0, 0);
  padding: 20px;
}

.orgchart table {
  border-spacing: 0;
  border-collapse: separate;
}

.orgchart > table:first-child {
  margin: 20px auto;
}

.orgchart td {
  text-align: center;
  vertical-align: top;
  padding: 0;
}

.orgchart .lines:nth-child(3) td {
  box-sizing: border-box;
  height: 20px;
}

.orgchart .lines .topLine {
  border-top: 2px solid #909399;
}

.orgchart .lines .rightLine {
  border-right: 1px solid #909399;
  float: none;
  border-radius: 0;
}

.orgchart .lines .leftLine {
  border-left: 1px solid #909399;
  float: none;
  border-radius: 0;
}

.orgchart .lines .downLine {
  background-color: #909399;
  margin: 0 auto;
  height: 20px;
  width: 2px;
  float: none;
}

/* node styling */
.orgchart .node {
  box-sizing: border-box;
  display: inline-block;
  position: relative;
  margin: 0;
  padding: 3px;
  border: 2px dashed transparent;
  text-align: center;
  width: 130px;
}

.orgchart.l2r .node,
.orgchart.r2l .node {
  width: 50px;
  height: 130px;
}

.orgchart .node:hover {
  background-color: #21b799;
  transition: 0.5s;
  cursor: default;
  z-index: 20;
}

.orgchart .node.focused {
  background-color: #21b799;
}

.orgchart .node.allowedDrop {
  border-color: rgba(68, 157, 68, 0.9);
}

.orgchart .node .title {
  text-align: center;
  font-size: 12px;
  font-weight: bold;
  height: 20px;
  line-height: 20px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  background-color: #909399;
  color: #fff;
  border-radius: 4px 4px 0 0;
}

.orgchart.b2t .node .title {
  transform: rotate(-180deg);
  transform-origin: center bottom;
}

.orgchart.l2r .node .title {
  transform: rotate(-90deg) translate(-40px, -40px) rotateY(180deg);
  transform-origin: bottom center;
  width: 120px;
}

.orgchart.r2l .node .title {
  transform: rotate(-90deg) translate(-40px, -40px);
  transform-origin: bottom center;
  width: 120px;
}

.orgchart .node .title .symbol {
  float: left;
  margin-top: 4px;
  margin-left: 2px;
}

.orgchart .node .content {
  box-sizing: border-box;
  width: 100%;
  height: 20px;
  font-size: 11px;
  line-height: 18px;
  border: 1px solid #909399;
  border-radius: 0 0 4px 4px;
  text-align: center;
  background-color: #fff;
  color: #333;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.orgchart.b2t .node .content {
  transform: rotate(180deg);
  transform-origin: center top;
}

.orgchart.l2r .node .content {
  transform: rotate(-90deg) translate(-40px, -40px) rotateY(180deg);
  transform-origin: top center;
  width: 120px;
}

.orgchart.r2l .node .content {
  transform: rotate(-90deg) translate(-40px, -40px);
  transform-origin: top center;
  width: 120px;
}

.orgchart .node {
  transition: transform 0.3s, opacity 0.3s;
}

.orgchart .node.is-active .title {
  background-color: #21b799 !important;
}

.orgchart .node.is-active .content {
  border: 1px solid #21b799 !important;
}
</style>
