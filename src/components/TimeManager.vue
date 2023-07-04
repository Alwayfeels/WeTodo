<template>
  <div>
    <canvas id="canvas" ref="canvasRef" class="canvas" :style="`width: ${layout.width}px; height: ${layout.height}px;`"
      :width="layout.width" :height="layout.height"></canvas>
    <div>{{ clockValue }}</div>
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
  clickAngle: null,
  isStartIconClick: false,
  isEndIconClick: false,
  dragBarStartAngle: 45,
  dragBarEndAngle: 190,
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
  drawDragBar(layout.dragBarStartAngleInit, layout.dragBarEndAngleInit)
  // drawCircleAtPosition({
  //   x: layout.centerX,
  //   y: layout.centerY,
  //   r: layout.drag_bar_R,
  //   angle: 45,
  //   circle_r: layout.drag_icon_size,
  // })
  // drawCircleAtPosition({
  //   x: layout.centerX,
  //   y: layout.centerY,
  //   r: layout.drag_bar_R,
  //   angle: 180,
  //   circle_r: layout.drag_icon_size,
  // })
  canvasDom = document.getElementById('canvas')
  addCanvasEventListener()
})

function addCanvasEventListener() {
  canvasDom.addEventListener('mousedown', (event) => {
    const { offsetX, offsetY } = event
    // 是否点击了dragbar图标
    const [isStartIconClick, isEndIconClick] = ([45, 180]).map(angle => {
      return isPointInCircle({
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
      console.log('click => ', state.clickAngle);
      canvasDom.addEventListener('mousemove', addCanvasMoveListener)
    }
  })
  canvasDom.addEventListener('mouseup', (event) => {
    canvasDom.removeEventListener('mousemove', addCanvasMoveListener)
  })
}

function addCanvasMoveListener(event) {
  const { offsetX, offsetY } = event
  let angle = calculateAngle({ x: layout.centerX, y: layout.centerY, pointX: offsetX, pointY: offsetY })
  console.log('move => ', Math.round(angle));
  drawDragBar(offsetX, offsetY)
}

function calculateAngle({ x, y, pointX, pointY }) {
  // 计算点击坐标相对于原点连线的角度
  var deltaX = pointX - x;
  var deltaY = y - pointY; // 注意这里的符号，因为 canvas 的坐标系与数学坐标系 y 轴方向相反
  var radian = Math.atan2(deltaY, deltaX);
  var angle = radian * (180 / Math.PI);
  // 将角度转换为正上方为0度
  angle = (angle + 360) % 360;
  return angle;
}

function drawClockBg() {
  drawBackground()
  drawImageOnArc(layout.centerX, layout.centerY, layout.bg_icon_R, 0, sparkles_icon, layout.bg_icon_size)
  drawImageOnArc(layout.centerX, layout.centerY, layout.bg_icon_R, 180, sun_icon, layout.bg_icon_size)
  drawScale()
}

function drawDragBar(startAngle = 0, endAngle = 90) {
  const {dragBarStartAngle, dragBarEndAngle} = state
  if (dragBarStartAngle === startAngle && dragBarEndAngle === endAngle) return
  state.dragBarStartAngle = startAngle
  state.dragBarEndAngle = endAngle
  clockValue.value = `${startAngle}°-${endAngle}°: ${getTimeFromAngle(startAngle)}-${getTimeFromAngle(endAngle)}`
  clearRing(layout.centerX, layout.centerY, layout.R1, layout.R2, colorConfig.dark)
  drawDragBarContent(layout.centerX, layout.centerY, layout.drag_bar_R, startAngle, endAngle, layout.drag_bar_width);
  drawLinesOnArc(layout.centerX, layout.centerY, layout.drag_bar_R, startAngle, endAngle, layout.drag_scale_length);
  drawImageOnArc(layout.centerX, layout.centerY, layout.drag_bar_R, startAngle, bed_icon, layout.drag_icon_size)
  drawImageOnArc(layout.centerX, layout.centerY, layout.drag_bar_R, endAngle, bell_icon, layout.drag_icon_size)
}

function drawCircleAtPosition({ x, y, r, angle, circle_r }) {
  angle -= 90
  // 转换为弧度
  angle = angle * Math.PI / 180;

  // 计算第二个圆的圆心坐标
  var secondX = x + Math.cos(angle) * r;
  var secondY = y + Math.sin(angle) * r;

  // 绘制圆
  ctx.beginPath();
  ctx.arc(secondX, secondY, circle_r, 0, 2 * Math.PI);
  ctx.fill();
}

function drawDragBarContent(x, y, r, startAngle, endAngle, arc_r) {
  ctx.save()
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

const drawBackground = () => {
  ctx.save()
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
function drawRing( x, y, r1, r2, color) {
  ctx.beginPath();
  ctx.arc(x, y, r2, 0, 2 * Math.PI);
  ctx.arc(x, y, r1, 0, 2 * Math.PI);
  ctx.closePath();

  ctx.lineWidth = r2 - r1;
  ctx.strokeStyle = color;
  ctx.stroke();
}

function clearRing(x, y, r1, r2, color) {
  ctx.beginPath();
  ctx.arc(x, y, r2, 0, 2 * Math.PI);
  ctx.arc(x, y, r1, 0, 2 * Math.PI, true);
  ctx.closePath();

  ctx.fillStyle = color;
  ctx.fill();
}

const drawLine = (start1, start2, end1, end2, color = '#000') => {
  ctx.beginPath();
  ctx.moveTo(start1, start2);
  ctx.lineTo(end1, end2);
  ctx.strokeStyle = color;
  ctx.stroke();
  ctx.closePath();
}

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

// 判断是否点击坐标是否在目标圆内
function isPointInCircle({ x, y, r, angle, circle_r, pointX, pointY }) {
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

const drawScale = () => {
  ctx.save();
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

</script>

<style lang="scss" scoped>
.canvas {
  image-rendering: high-quality;
  
}
</style>