<template>
  <div>
    <canvas id="canvas" ref="canvasRef" class="canvas" :style="`width: ${layout.width}px; height: ${layout.height}px;`"
      :width="layout.width" :height="layout.height"></canvas>
    <div>{{ clockValue }}</div>
    <div>点击角度 {{ state.clickAngle }}</div>
    <div>当前角度 {{ state.moveAngle }}</div>
    <div>转动角度差 {{ state.dragOffsetAngle }}</div>
    <div>开始角度 {{ state.dragBarStartAngle }}</div>
    <div>结束角度 {{ state.dragBarEndAngle }}</div>
    <div>isStartIconClick {{ state.isStartIconClick }}</div>
    <div>isEndIconClick {{ state.isEndIconClick }}</div>
    
  </div>
</template>

<script setup>
import { ref, h, computed, nextTick, watch, reactive, toRaw, onMounted } from "vue";
import bed_icon from '../assets/bed.png'
import bell_icon from '../assets/bell.png'
import sparkles_icon from '../assets/sparkles.png'
import sun_icon from '../assets/sun.png'

const layout = reactive({
  width: 300,
  height: 300,
  R1: 150,
  R2: 100,
  scale_R: 95,
  scale_font_size: 12,
  scale_number_R: 75,
  bg_icon_R: 50,
  bg_icon_size: 26,
  dragBarStartAngleInit: 45,
  dragBarEndAngleInit: 180,
  drag_icon_size: 20,
  drag_scale_length: 18,
  drag_bar_R: 125,
  drag_bar_width: 20,
  drag_bar_scale_width: 20,
  centerX: computed(() => layout.width / 2),
  centerY: computed(() => layout.height / 2),
});
const colorConfig = reactive({
  light: '#ffffff',
  shallowlight: '#8e8d92',
  shallowDark: '#2c2c2e',
  dark: '#010101',
  night: '#5edfde',
  daylight: '#f7d500'
})
const state = reactive({
  clickAngle: 0, // 点击初始角度，偏移量判断依据
  moveAngle: 0, // 当前拖动角度
  dragOffsetAngle: 0, // 点击角度和开始角度差,
  isStartIconClick: false, // 是否点击了头部图标
  isEndIconClick: false, // 是否点击了尾部图标
  dragBarStartAngle: 45, // 拖拽条 开始角度
  dragBarEndAngle: 180, // 拖拽条 结束角度
})

let canvasDom
const canvasRef = ref(null)
let ctx = ref()
const canvas = reactive({
  instance: canvasRef,
  context: null,
});

let clockValue = ref();

/**
 * @desc: 初始化
 */
onMounted(() => {
  ctx = canvas.instance?.getContext('2d')
  canvas.context = ctx
  clockValue.value = canvas.context ? '无数据' : '不支持Canvas'
  drawClockBg()
  ctx.save()
  drawDragBar(layout.dragBarStartAngleInit, layout.dragBarEndAngleInit)
  drawCircleAtPosition({
    x: layout.centerX,
    y: layout.centerY,
    r: layout.drag_bar_R,
    angle: state.dragBarStartAngle,
    circle_r: layout.drag_icon_size,
  })
  drawCircleAtPosition({
    x: layout.centerX,
    y: layout.centerY,
    r: layout.drag_bar_R,
    angle: state.dragBarEndAngle,
    circle_r: layout.drag_icon_size,
  })
  canvasDom = document.getElementById('canvas')
  addCanvasEventListener()
})

function addCanvasEventListener() {
  canvasDom.addEventListener('mousedown', (event) => {
    const { offsetX, offsetY } = event
    // 是否点击了dragbar
    // const isClickDragBar = isPointInDragBar({
    //   x: layout.centerX,
    //   y: layout.centerY,
    //   r: layout.drag_bar_R,
    //   startAngle: state.dragBarStartAngle,
    //   endAngle: state.dragBarEndAngle,
    //   lineWidth: layout.drag_bar_width,
    //   pointX: offsetX,
    //   pointY: offsetY
    // })
    // if (isClickDragBar) {
    //   console.log('rotate  eeeeeeeeee');
    //   rotateDragBar(45)
    // }
    // console.log('isClickDragBar ??? => ', isClickDragBar);
    // 是否点击了dragbar图标
    const [isStartIconClick, isEndIconClick] = ([state.dragBarStartAngle, state.dragBarEndAngle]).map(angle => {
      return isClickInCircle({
        x: layout.centerX,
        y: layout.centerY,
        r: layout.drag_bar_R,
        angle: angle,
        circle_r: layout.drag_icon_size,
        pointX: offsetX,
        pointY: offsetY
      })
    })
    state.isStartIconClick = isStartIconClick
    state.isEndIconClick = isEndIconClick
    if (isStartIconClick || isEndIconClick) {
      state.clickAngle = Math.round(calculateAngle({ x: layout.centerX, y: layout.centerY, pointX: offsetX, pointY: offsetY }))
      state.dragOffsetAngle = state.clickAngle - state.dragBarStartAngle
      canvasDom.addEventListener('mousemove', onCanvasMouseMove)
    }
  })
  canvasDom.addEventListener('mouseup', (event) => {
    state.isStartIconClick = false
    state.isEndIconClick = false
    state.currAngle = 0
    state.clickAngle = 0
    canvasDom.removeEventListener('mousemove', onCanvasMouseMove)
  })
}

/**
 * @desc: 核心
 */
function onCanvasMouseMove(event) {
  const { offsetX, offsetY } = event
  let currAngle = Math.round(calculateAngle({ x: layout.centerX, y: layout.centerY, pointX: offsetX, pointY: offsetY }))
  if (state.moveAngle === currAngle) return
  state.moveAngle = currAngle
  const newStartAngle = state.isStartIconClick ? Math.round(currAngle + state.dragOffsetAngle): state.dragBarStartAngle 
  const newEndAngle = state.isEndIconClick ? Math.round(currAngle - currAngle + state.dragOffsetAngle): state.dragBarEndAngle
  console.log('拖拽偏转角度 => ', state.dragOffsetAngle);

  state.dragBarStartAngle = newStartAngle
  state.dragBarEndAngle = newEndAngle
  console.log('重绘角度 => ', newStartAngle, newEndAngle);
  drawDragBar(newStartAngle, newEndAngle)
}

function calculateAngle({ x, y, pointX, pointY }) {
  var dx = pointX - x;
  var dy = y - pointY;
  var angle = Math.atan2(dx, dy) * (180 / Math.PI);
  if (angle < 0) {
    angle = 360 + angle;
  } else if (angle >= 360) {
    angle = angle % 360;
  }
  return angle;
}
console.log('.....', calculateAngle({ x: 10, y: 10, pointX: 10, pointY: 20 }));

function drawClockBg() {
  drawBackground()
  drawImageOnArc(layout.centerX, layout.centerY, layout.bg_icon_R, 0, sparkles_icon, layout.bg_icon_size)
  drawImageOnArc(layout.centerX, layout.centerY, layout.bg_icon_R, 180, sun_icon, layout.bg_icon_size)
  drawScale()
}

function rotateDragBar(angle) {
  ctx.restore()
  const perAngle = Math.PI / 180
  ctx.rotate(angle * perAngle); // 45度角
}

function drawDragBar(startAngle = 0, endAngle = 90) {
  if (startAngle >= 360 || endAngle >= 360) {
    startAngle = startAngle % 360
    endAngle = endAngle % 360
  }
  clearRing(layout.centerX, layout.centerY, (layout.drag_bar_R - layout.drag_bar_width), (layout.drag_bar_R + layout.drag_bar_width))
  clockValue.value = `${startAngle}°-${endAngle}°: ${getTimeFromAngle(startAngle)}-${getTimeFromAngle(endAngle)}`
  drawDragBarContent(layout.centerX, layout.centerY, layout.drag_bar_R, startAngle, endAngle, layout.drag_bar_width);
  drawLinesOnArc(layout.centerX, layout.centerY, layout.drag_bar_R, startAngle, endAngle, layout.drag_scale_length);
  drawImageOnArc(layout.centerX, layout.centerY, layout.drag_bar_R, startAngle, bed_icon, layout.drag_icon_size)
  drawImageOnArc(layout.centerX, layout.centerY, layout.drag_bar_R, endAngle, bell_icon, layout.drag_icon_size)
}

function drawCircleAtPosition({ x, y, r, angle, circle_r }) {
  // 转换为弧度
  const arc = (angle - 90) * Math.PI / 180;

  // 计算第二个圆的圆心坐标
  var secondX = x + Math.cos(arc) * r;
  var secondY = y + Math.sin(arc) * r;

  // 绘制圆
  ctx.beginPath();
  ctx.arc(secondX, secondY, circle_r, 0, 2 * Math.PI);
  ctx.fillStyle = colorConfig.light
  ctx.fill();
}

/**
 * @desc: 操作条
 */

function drawDragBarContent(x, y, r, startAngle, endAngle, arc_r) {
  // 转换为弧度
  startAngle = (startAngle - 90) * Math.PI / 180;
  endAngle = (endAngle - 90) * Math.PI / 180;

  ctx.beginPath();
  ctx.lineWidth = arc_r * 2; // 设置线条宽度为 arc_r 的两倍
  ctx.lineCap = 'round'; // 设置线条末端样式为圆形
  ctx.arc(x, y, r, startAngle, endAngle);
  ctx.strokeStyle = colorConfig.shallowDark;
  ctx.stroke();
}

function isPointInDragBar({ x, y, r, startAngle, endAngle, lineWidth, pointX, pointY }) {
  var dx = pointX - x;
  var dy = pointY - y;
  var distance = Math.sqrt(dx * dx + dy * dy);

  if (distance < r + lineWidth && distance > r - lineWidth) {
    var angle = Math.atan2(dy, dx);
    angle = (angle < 0) ? (angle + 2 * Math.PI) : angle;
    // console.log('condition1 success; angle=>', angle);
    return true
    if (startAngle > endAngle) {
      return angle >= startAngle || angle <= endAngle;
    } else {
      return angle >= startAngle && angle <= endAngle;
    }
  }
  return false;
}

// 绘制图标
function drawImageOnArc(x, y, r, angle, image, imageSize) {
  // 转换为弧度
  angle = (angle - 90) * Math.PI / 180;

  // 计算图像起点的坐标
  var imageX = x + Math.cos(angle) * r - imageSize / 2;
  var imageY = y + Math.sin(angle) * r - imageSize / 2;

  // 绘制图像
  if (typeof image === 'string') {
    var img = new Image();
    img.src = image;
    img.onload = function () {
      ctx.drawImage(img, imageX, imageY, imageSize, imageSize);
    };
  } else {
    ctx.drawImage(image, imageX, imageY, imageSize, imageSize);
  }
}


// 绘制拖动条纹路
function drawLinesOnArc(x, y, r, startAngle, endAngle, lineLength, angleOffset = 7) {
  r = r - lineLength / 2;
  // 转换为弧度
  startAngle = (startAngle + angleOffset - 90) * Math.PI / 180; // 偏转7度以容纳起止图标
  endAngle = (endAngle - angleOffset - 90) * Math.PI / 180;

  // 计算直线起点和终点的坐标
  var startX = x + Math.cos(startAngle) * (r + lineLength);
  var startY = y + Math.sin(startAngle) * (r + lineLength);
  var endX = x + Math.cos(startAngle) * r;
  var endY = y + Math.sin(startAngle) * r;

  // 设置线条样式
  ctx.strokeStyle = colorConfig.shallowlight;
  ctx.lineWidth = 1;

  // 逐度绘制直线
  for (var angle = startAngle + 1 * Math.PI / 180; angle <= endAngle; angle += 2 * Math.PI / 180) {
    startX = x + Math.cos(angle) * (r + lineLength);
    startY = y + Math.sin(angle) * (r + lineLength);
    endX = x + Math.cos(angle) * r;
    endY = y + Math.sin(angle) * r;

    ctx.beginPath();
    ctx.moveTo(startX, startY);
    ctx.lineTo(endX, endY);
    ctx.stroke();
  }
}

/**
 * @desc: 背景绘制
 */
// 基础背景
const drawBackground = () => {
  ctx.beginPath();
  ctx.arc(layout.centerX, layout.centerY, layout.R1, 0, Math.PI * 2);
  ctx.fillStyle = colorConfig.shallowDark;
  ctx.fill
  ctx.fill()
  ctx.moveTo(layout.centerX + layout.R2, layout.centerY);
  ctx.arc(layout.centerX, layout.centerY, layout.R2, 0, Math.PI * 2, true);
  ctx.fillStyle = colorConfig.dark;
  ctx.fill()
  ctx.restore()
}

// 绘制圆环
function clearRing(x, y, r1, r2, color = colorConfig.dark) {
  ctx.beginPath();
  ctx.arc(x, y, r2, 0, 2 * Math.PI);
  ctx.arc(x, y, r1, 0, 2 * Math.PI, true);
  ctx.closePath();

  ctx.fillStyle = color;
  ctx.fill();
}

// 绘制线条
const drawLine = (start1, start2, end1, end2, color = '#000') => {
  ctx.beginPath();
  ctx.moveTo(start1, start2);
  ctx.lineTo(end1, end2);
  ctx.strokeStyle = color;
  ctx.stroke();
  ctx.closePath();
}

// 判断是否点击坐标是否在目标圆内
function isClickInCircle({ x, y, r, angle, circle_r, pointX, pointY }) {
  ctx.beginPath();
  // 转换为弧度
  angle = (angle - 90) * Math.PI / 180;
  // 计算第二个圆的圆心坐标
  var secondX = x + Math.cos(angle) * r;
  var secondY = y + Math.sin(angle) * r;
  // 保存绘制圆的路径
  ctx.arc(secondX, secondY, circle_r, 0, 2 * Math.PI);
  // 判断点是否在路径内
  return ctx.isPointInPath(pointX, pointY);
}

// 计算时间
function getTimeFromAngle(angle) {
  // 将角度转换为0-360范围内
  angle = angle % 360;
  if (angle < 0) {
    angle += 360;
  }

  // 计算小时和分钟
  var totalMinutes = Math.round(angle / 360 * 24 * 60);
  var hours = Math.floor(totalMinutes / 60);
  var minutes = Math.round((totalMinutes % 60) / 5) * 5;

  // 格式化小时和分钟
  var formattedHours = hours.toString().padStart(2, '0');
  var formattedMinutes = minutes.toString().padStart(2, '0');

  return formattedHours + ':' + formattedMinutes;
}

// 绘制时钟表盘刻度
const drawScale = () => {
  // 画制时钟刻度
  // 定义1°
  var pi = Math.PI * 2 / 360;
  for (var i = 0; i < (24 * 4); i++) {
    var x = -Math.cos(pi * (360 / (24 * 4)) * i);
    var y = Math.sin(pi * (360 / (24 * 4)) * i);
    if (i % 8 == 0) {
      // 绘制时钟数字
      ctx.lineWidth = 1;
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.font = `bold ${layout.scale_font_size}px 微软雅黑`;
      ctx.fillStyle = i % 24 === 0 ? colorConfig.light : colorConfig.shallowlight;
      ctx.fillText(i / 4, layout.centerX + layout.scale_number_R * y, layout.centerX + layout.scale_number_R * x);
    }
    ctx.lineWidth = 0.8;
    const lineLength = i % 4 == 0 ? 6 : 3;
    drawLine(layout.centerX + layout.scale_R * x, layout.centerY + layout.scale_R * y, layout.centerX + (layout.scale_R - lineLength) * x, layout.centerX + (layout.scale_R - lineLength) * y, colorConfig.shallowlight)
  }
}

</script>

<style lang="scss" scoped>
.canvas {
  image-rendering: high-quality;

}
</style>