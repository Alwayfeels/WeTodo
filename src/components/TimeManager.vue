<template>
  <div style="display: flex; width: 60vw;">
    <div style="width: 20vw; display: flex; align-items: flex-start; flex-direction: column;">
      <div style="font-weight: bold;">配置参数</div>
      <div>
        <div v-for="key in Object.keys(layout)" :key="key" style="text-align: left;">{{ key }}: {{ layout[key] }}</div>
      </div>
    </div>
    <div style="flex: 1">
      <canvas id="canvas" ref="canvasRef" class="canvas" :style="`width: ${layout.width}px; height: ${layout.height}px;`"
        :width="layout.width" :height="layout.height"></canvas>
      <div>{{ clockValue }}</div>
      <div>点击角度 {{ state.clickAngle }}°</div>
      <div>当前角度 {{ state.mouseAngle }}°</div>
      <div>实时开始角度 {{ state.dragBarStartAngle }}°</div>
      <div>实时结束角度 {{ state.dragBarEndAngle }}°</div>
      <div>点击时开始角度 {{ state.oldDragBarStartAngle }}°</div>
      <div>点击时结束角度 {{ state.oldDragBarEndAngle }}°</div>
      <div>拖动条弧度 {{ state.dragBarAngle }}°</div>
      <div>拖动条弧度最短距离 {{ layout.dragBarMinAngle }}°</div>
      <div>拖动条弧度最长距离 {{ layout.dragBarMaxAngle }}°</div>
      <div>拖动条移动弧度 {{ state.dragMoveAngle }}°</div>
      <div>isStartIconClick {{ state.isStartIconClick }}</div>
      <div>isEndIconClick {{ state.isEndIconClick }}</div>
      <div>isDragBarClick {{ state.isDragBarClick }}</div>
      <div>imageLoaded {{ state.imageLoaded }}</div>
    </div>
  </div>
</template>

<script setup>
import { ref, h, computed, nextTick, watch, reactive, onBeforeMount, onMounted } from "vue";

/**
 * @desc: 配置参数
 */
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
  dragBarStartAngleInit: 90,
  dragBarEndAngleInit: 270,
  drag_icon_size: 20,
  drag_scale_length: 15,
  drag_bar_R: 125,
  drag_bar_width: 20,
  drag_bar_scale_width: 20,
  centerX: computed(() => layout.width / 2),
  centerY: computed(() => layout.height / 2),
  dragBarMinAngle: 20,
  dragBarMaxAngle: 300
});
const colorConfig = reactive({
  light: '#ffffff',
  shallowlight: '#8e8d92',
  shallowDark: '#2c2c2e',
  dark: '#010101',
  night: '#5edfde',
  daylight: '#f7d500'
})
const images = {}

// 状态
const state = reactive({
  imageLoaded: false,
  clickAngle: 0, // 点击初始角度，偏移量判断依据
  mouseAngle: 0, // 当前拖动角度
  dragMoveAngle: 0, // 鼠标离点击点拖动的角度
  isStartIconClick: false, // 是否点击了头部图标
  isEndIconClick: false, // 是否点击了尾部图标
  isDragBarClick: false, // 是否点击了拖拽条
  dragBarStartAngle: layout.dragBarStartAngleInit, // 拖拽条 开始角度
  dragBarEndAngle: layout.dragBarEndAngleInit, // 拖拽条 结束角度
  oldDragBarStartAngle: layout.dragBarStartAngleInit, // 旧的拖拽条 开始角度
  oldDragBarEndAngle: layout.dragBarEndAngleInit, // 旧的拖拽条 结束角度
  // 拖拽条的弧长
  dragBarAngle: computed(() => state.dragBarStartAngle > state.dragBarEndAngle ? 360 - state.dragBarStartAngle + state.dragBarEndAngle : state.dragBarEndAngle - state.dragBarStartAngle),
})

// 静态加载图片
function loadImages() {
  const imagesUrl = {
    bed_icon: '/public/bed.png',
    bell_icon: '/public/bell.png',
    sparkles_icon: '/public/sparkles.png',
    sun_icon: '/public/sun.png',
  }
  let loadSuccessNum = 0;
  for (let name of Object.keys(imagesUrl)) {
    const imgSrc = imagesUrl[name]
    const imageUrl = new URL(imgSrc, import.meta.url).href
    let image = new Image();
    image.src = imageUrl
    images[name] = image
    image.onload = () => {
      loadSuccessNum++
      console.log('loadSuccessNum>>>', loadSuccessNum);
      if (loadSuccessNum === Object.keys(imagesUrl).length) {
        nextTick(() => {
          console.log('images>>>', images);
          state.imageLoaded = true
        })
      }
    }
  }
}

let clockValue = ref(); // show string

let canvasDom = null
let cacheCanvasDom = null
const canvasRef = ref(null) // dom
let ctx = ref() // 绘制实例
const canvasCtx = reactive({
  render: null, // 渲染ctx
  cache: null, // 缓存ctx
});

onBeforeMount(() => {
  loadImages()
})

watch(() => state.imageLoaded, (val) => {
  if (val) {
    init()
  }
})

/**
 * @desc: init
 */

 function init() {
  initCacheCanvas()
  canvasDom = canvasRef.value
  canvasCtx.render = canvasDom.getContext('2d')
  console.log('canvasCtx.render', canvasCtx);
  // clockValue.value = canvas.context ? '无数据' : '不支持Canvas'
  drawClockBg()
  drawDragBar(layout.dragBarStartAngleInit, layout.dragBarEndAngleInit)
  renderFromCache()
  addCanvasEventListener()
 }

function renderFromCache() {
  canvasCtx.render.clearRect(0, 0, layout.width, layout.height);
  canvasCtx.render.drawImage(cacheCanvasDom, 0, 0)
}

function initCacheCanvas() {
  cacheCanvasDom = document.createElement('canvas')
  cacheCanvasDom.width = layout.width
  cacheCanvasDom.height = layout.height
  canvasCtx.cache = cacheCanvasDom.getContext('2d');
  canvasCtx.cache.width = layout.width
  canvasCtx.cache.height = layout.height
  ctx = canvasCtx.cache
}
/**
 * @desc: event listener
 */
function addCanvasEventListener() {
  canvasDom.addEventListener('mousedown', onCanvasMouseDown)
  canvasDom.addEventListener('touchstart', onCanvasMouseDown)

  canvasDom.addEventListener('mouseup', onCanvasMouseUp)
  canvasDom.addEventListener('touchend', onCanvasMouseUp)
}

async function onCanvasMouseUp(event) {
  event.preventDefault()
  state.isStartIconClick = false
  state.isEndIconClick = false
  state.isDragBarClick = false
  await nextTick()
  state.currAngle = 0
  state.clickAngle = 0
  state.dragMoveAngle = 0
  canvasDom.removeEventListener('mousemove', onCanvasMouseMove)
  canvasDom.removeEventListener('touchmove', onCanvasMouseMove)
}

function onCanvasMouseDown(event) {
    event.preventDefault()
    const { offsetX, offsetY } = event
    console.log('//////////////////', event);
    // 是否点击了dragbar上的图标
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
    // 是否点击了dragBar本身
    const isDragBarClick = isPointInArcZone({
      x: layout.centerX,
      y: layout.centerY,
      r1: layout.drag_bar_R - layout.drag_bar_width,
      r2: layout.drag_bar_R + layout.drag_bar_width,
      startAngle: state.dragBarStartAngle,
      endAngle: state.dragBarEndAngle,
      pointX: offsetX,
      pointY: offsetY
    })
    state.isStartIconClick = isStartIconClick
    state.isEndIconClick = isEndIconClick
    state.isDragBarClick = isDragBarClick
    if (isStartIconClick || isEndIconClick || isDragBarClick) {
      state.clickAngle = Math.round(calculateAngle({ x: layout.centerX, y: layout.centerY, pointX: offsetX, pointY: offsetY }))
      state.oldDragBarStartAngle = formatAngle(state.dragBarStartAngle)
      state.oldDragBarEndAngle = formatAngle(state.dragBarEndAngle)
      canvasDom.addEventListener('mousemove', onCanvasMouseMove)
      canvasDom.addEventListener('touchmove', onCanvasMouseMove)
      return;
    }
    renderFromCache()
  }

/**
 * @desc: event handler
 */
async function onCanvasMouseMove(event) {
  event.preventDefault()
  const { offsetX, offsetY } = event
  let currAngle = Math.round(calculateAngle({ x: layout.centerX, y: layout.centerY, pointX: offsetX, pointY: offsetY }))
  if (state.mouseAngle === currAngle) return
  state.mouseAngle = currAngle // 鼠标的角度
  state.dragMoveAngle = currAngle - state.clickAngle // 拖动过的角度
  if (state.isStartIconClick) {
    state.dragBarStartAngle = formatAngle(state.dragMoveAngle + state.oldDragBarStartAngle) // 新的开始角度
  }
  else if (state.isEndIconClick) {
    state.dragBarEndAngle = formatAngle(state.dragMoveAngle + state.oldDragBarEndAngle) // 新的结束角度
  }
  else if (state.isDragBarClick) {
    state.dragBarStartAngle = formatAngle(state.dragMoveAngle + state.oldDragBarStartAngle) // 新的开始角度
    state.dragBarEndAngle = formatAngle(state.dragMoveAngle + state.oldDragBarEndAngle) // 新的结束角度
  }
  // dragbar length min/max limit
  if (state.dragBarAngle <= layout.dragBarMinAngle) {
    if (state.isStartIconClick) {
      state.dragBarEndAngle = formatAngle(state.dragBarStartAngle + layout.dragBarMinAngle)
    }
    if (state.isEndIconClick) {
      state.dragBarStartAngle = formatAngle(state.dragBarEndAngle - layout.dragBarMinAngle)
    }
  }
  if (state.dragBarAngle >= layout.dragBarMaxAngle) {
    if (state.isStartIconClick) {
      state.dragBarEndAngle = formatAngle(state.dragBarStartAngle + layout.dragBarMaxAngle)
    }
    if (state.isEndIconClick) {
      state.dragBarStartAngle = formatAngle(state.dragBarEndAngle - layout.dragBarMaxAngle)
    }
  }
  drawDragBar(formatAngle(state.dragBarStartAngle), formatAngle(state.dragBarEndAngle))
  renderFromCache()
}

function drawClockBg() {
  drawBackground()
  drawImageOnArc(layout.centerX, layout.centerY, layout.bg_icon_R, 0, images.sparkles_icon, layout.bg_icon_size)
  drawImageOnArc(layout.centerX, layout.centerY, layout.bg_icon_R, 180, images.sun_icon, layout.bg_icon_size)
  drawScale()
}

function drawDragBar(startAngle = 0, endAngle = 90) {
  startAngle = formatAngle(startAngle)
  endAngle = formatAngle(endAngle)
  clockValue.value = `${startAngle}°-${endAngle}°: ${getTimeFromAngle(startAngle)}-${getTimeFromAngle(endAngle)}`
  drawArc({ x: layout.centerX, y: layout.centerY, r: layout.drag_bar_R, startAngle, endAngle, lineWidth: layout.drag_bar_width * 2 });
  drawLinesOnArc({ x: layout.centerX, y: layout.centerY, r: layout.drag_bar_R, startAngle, lineLength: layout.drag_scale_length });
  drawArc({
    x: layout.centerX,
    y: layout.centerY,
    r: layout.drag_bar_R,
    lineWidth: layout.drag_bar_width * 2,
    startAngle: state.dragBarEndAngle,
    endAngle: state.dragBarStartAngle,
    color: colorConfig.dark
  })
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
  drawImageOnArc(layout.centerX, layout.centerY, layout.drag_bar_R, startAngle, images.bed_icon, layout.drag_icon_size)
  drawImageOnArc(layout.centerX, layout.centerY, layout.drag_bar_R, endAngle, images.bell_icon, layout.drag_icon_size)
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
  ctx.fillStyle = colorConfig.shallowDark
  ctx.fill();
}

/**
 * @desc: 操作条
 */

function drawArc({ x, y, r, startAngle, endAngle, lineWidth, color = colorConfig.shallowDark, lineCap = 'butt' }) {
  // 转换为弧度
  startAngle = (startAngle - 90) * Math.PI / 180;
  endAngle = (endAngle - 90) * Math.PI / 180;

  ctx.beginPath();
  ctx.lineWidth = lineWidth; // 设置线条宽度为 arc_r 的两倍
  ctx.lineCap = lineCap;
  ctx.arc(x, y, r, startAngle, endAngle);
  ctx.strokeStyle = color;
  ctx.stroke();
}

// 绘制拖动条纹路
function drawLinesOnArc({ x, y, r, startAngle = 0, lineLength, lineWidth = 1.8, lineGap = 3.2 }) {
  ctx.beginPath();
  ctx.closePath();

  for (var angle = startAngle; angle < startAngle + 360; angle += lineGap) {
    var radians = angle * Math.PI / 180;
    var startX = x + (r - lineLength / 2) * Math.sin(radians);
    var startY = y - (r - lineLength / 2) * Math.cos(radians);
    var endX = x + (r + lineLength / 2) * Math.sin(radians);
    var endY = y - (r + lineLength / 2) * Math.cos(radians);

    ctx.moveTo(startX, startY);
    ctx.lineTo(endX, endY);
  }
  ctx.lineWidth = lineWidth
  ctx.strokeStyle = colorConfig.shallowlight
  ctx.stroke(); // 绘制刻度线
  // 返回绘制结果
  return canvas;
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
      renderFromCache()
    };
  } else {
    ctx.drawImage(image, imageX, imageY, imageSize, imageSize);
    renderFromCache()
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

// 绘制线条
const drawLine = (start1, start2, end1, end2, color = '#000') => {
  ctx.beginPath();
  ctx.moveTo(start1, start2);
  ctx.lineTo(end1, end2);
  ctx.strokeStyle = color;
  ctx.stroke();
  ctx.closePath();
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

/**
 * @desc: 判断落点函数 ============================
 */
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
  ctx.closePath();
  // 判断点是否在路径内
  return ctx.isPointInPath(pointX, pointY);
}

// 判断落点是否在目标扇形区域内
function isPointInArcZone({ x, y, r1, r2, startAngle, endAngle, pointX, pointY }) {
  // 转换为弧度
  startAngle = (startAngle - 90) * Math.PI / 180;
  endAngle = (endAngle - 90) * Math.PI / 180;

  ctx.beginPath();
  ctx.moveTo(x + r1 * Math.cos(startAngle), y + r1 * Math.sin(startAngle));
  ctx.arc(x, y, r2, startAngle, endAngle);
  ctx.lineTo(x + r1 * Math.cos(endAngle), y + r1 * Math.sin(endAngle));
  ctx.arc(x, y, r1, endAngle, startAngle, true);
  ctx.closePath();
  return ctx.isPointInPath(pointX, pointY);
}

/**
 * @desc: 辅助计算函数 ===============
 */

// 格式化度数，始终为0-360
function formatAngle(angle) {
  while (angle < 0 || angle >= 360) {
    if (angle < 0) {
      angle += 360;
    } else if (angle >= 360) {
      angle -= 360;
    }
  }
  return angle;
}

// calc angle by 2 point(top = 0°)
function calculateAngle({ x, y, pointX, pointY }) {
  var dx = pointX - x;
  var dy = y - pointY;
  var angle = Math.atan2(dx, dy) * (180 / Math.PI);
  return formatAngle(angle);
}

// 计算时间
function getTimeFromAngle(angle) {
  // 将角度转换为0-360范围内
  angle = formatAngle(angle)

  // 计算小时和分钟
  var totalMinutes = Math.round(angle / 360 * 24 * 60);
  var hours = Math.floor(totalMinutes / 60);
  var minutes = Math.round((totalMinutes % 60) / 5) * 5;

  // 格式化小时和分钟
  var formattedHours = hours.toString().padStart(2, '0');
  var formattedMinutes = minutes.toString().padStart(2, '0');

  return formattedHours + ':' + formattedMinutes;
}

</script>

<style lang="scss" scoped></style>