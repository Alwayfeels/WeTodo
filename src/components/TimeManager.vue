<template>
  <div>
    <canvas ref="canvasRef" class="canvas" :style="`width: ${layout.width}px; height: ${layout.height}px;`"
      :width="layout.width" :height="layout.height"></canvas>
    <div>{{ supportState }}</div>
  </div>
</template>

<script setup>
import { ref, h, computed, nextTick, watch, reactive, toRaw, onMounted } from "vue";

const layout = reactive({
  width: 300,
  height: 300,
  R1: 150,
  R2: 100,
  scale_R: 95,
  scale_number_R: 75,
  icon_size: 30,
});
const R = layout.width / 2;

const colorConfig = reactive({
  light: '#ccc',
  shallowlight: '#999',
  shallowDark: '#666',
  dark: '#333',
  night: '#5edfde',
  daylight: '#f7d500'
})

const canvasRef = ref(null)
let ctx = ref()
const canvas = reactive({
  instance: canvasRef,
  context: null,
});

let supportState = ref();
onMounted(() => {
  ctx = canvas.instance?.getContext('2d')
  canvas.context = ctx
  supportState.value = canvas.context ? '支持Canvas' : '不支持Canvas'
  drawBackground()
  drawScale()
  drawSunlightIcon(5000 - 500, 6000, layout.icon_size) // icon=1000x1000 
  drawSparklesIcon(0, 0, layout.icon_size) // icon=1000x1000 
})

const drawBackground = () => {
  ctx.save()
  ctx.beginPath();
  ctx.arc(layout.width / 2, layout.height / 2, layout.R1, 0, Math.PI * 2);
  ctx.fillStyle = colorConfig.shallowDark;
  ctx.fill()
  ctx.moveTo(layout.width / 2 + layout.R2, layout.height / 2);
  ctx.arc(layout.width / 2, layout.height / 2, layout.R2, 0, Math.PI * 2, true);
  ctx.fillStyle = colorConfig.dark;
  ctx.fill()
  ctx.restore()
}

const drawLine = (start1, start2, end1, end2, color = '#000') => {
  ctx.beginPath();
  ctx.moveTo(start1, start2);
  ctx.lineTo(end1, end2);
  ctx.strokeStyle = color;
  ctx.stroke();
  ctx.closePath();
}

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
      ctx.font = 'bold 12px 微软雅黑';
      ctx.fillStyle = i % 24 === 0 ? colorConfig.light : colorConfig.shallowlight;
      ctx.fillText(i / 4, R + layout.scale_number_R * y, R + layout.scale_number_R * x);
    }
    ctx.lineWidth = 0.8;
    const lineLength = i % 4 == 0 ? 6 : 3;
    drawLine(R + layout.scale_R * x, R + layout.scale_R * y, R + (layout.scale_R - lineLength) * x, R + (layout.scale_R - lineLength) * y, colorConfig.shallowlight)
  }
}

const drawSparklesIcon = (offsetX, offsetY, size = 100) => {
  const offsetObj = { offsetX, offsetY }
  ctx.save();
  ctx.miterLimit = 4;
  const scale = size / 1000
  // ctx.scale(scale, scale);
  ctx.scale(0.01, 0.01);
  ctx.save();
  ctx.fillStyle = colorConfig.night;  //填充颜色
  ctx.beginPath();
  offsetMoveTo(offsetObj, 488.009143, 231.497143);
  offsetBezierCurveTo(offsetObj, 493.129143, 231.497143, 495.707429, 228.498286, 497.005714, 223.78057099999998);
  offsetBezierCurveTo(offsetObj, 510.281143, 152.210285, 509.44, 150.49142799999998, 584.009143, 136.35657099999997);
  offsetBezierCurveTo(offsetObj, 589.129143, 135.49714199999997, 592.146286, 132.51657099999997, 592.146286, 127.35999999999997);
  offsetBezierCurveTo(offsetObj, 592.146286, 122.22171399999998, 589.147429, 119.20457099999997, 583.990857, 118.36342899999997);
  offsetBezierCurveTo(offsetObj, 509.860571, 103.36914299999997, 512, 101.65028599999997, 497.005714, 30.921142999999972);
  offsetBezierCurveTo(offsetObj, 495.72571400000004, 26.22171399999997, 493.147428, 23.222856999999973, 488.009143, 23.222856999999973);
  offsetBezierCurveTo(offsetObj, 482.852572, 23.222856999999973, 480.292572, 26.221713999999974, 478.994286, 30.921142999999972);
  offsetBezierCurveTo(offsetObj, 464, 101.65028599999997, 466.56, 103.35085699999998, 391.990857, 118.36342899999997);
  offsetBezierCurveTo(offsetObj, 387.291428, 119.20457199999997, 383.853714, 122.20342899999997, 383.853714, 127.35999999999997);
  offsetBezierCurveTo(offsetObj, 383.853714, 132.49828599999998, 387.29142800000005, 135.49714299999997, 391.990857, 136.35657099999997);
  offsetBezierCurveTo(offsetObj, 466.578286, 151.35085699999996, 465.718857, 152.21028499999997, 478.994286, 223.78057099999998);
  offsetBezierCurveTo(offsetObj, 480.27428599999996, 228.49828499999998, 482.852572, 231.49714199999997, 487.990857, 231.497143);
  ctx.closePath();
  offsetMoveTo(offsetObj, 280.576, 526.354286);
  offsetBezierCurveTo(offsetObj, 288.713143, 526.354286, 294.29028600000004, 521.216, 295.149714, 513.499429);
  offsetBezierCurveTo(offsetObj, 310.564571, 399.06742899999995, 314.422857, 399.06742899999995, 432.71314300000006, 376.356571);
  offsetBezierCurveTo(offsetObj, 440.42971400000005, 375.076571, 446.0068570000001, 369.91999999999996, 446.0068570000001, 361.782857);
  offsetBezierCurveTo(offsetObj, 446.0068570000001, 354.066286, 440.4297140000001, 348.507428, 432.71314300000006, 347.209143);
  offsetBezierCurveTo(offsetObj, 314.4228570000001, 330.93485699999997, 310.14400000000006, 327.076572, 295.149714, 210.50514299999998);
  offsetBezierCurveTo(offsetObj, 294.29028500000004, 202.788572, 288.713143, 197.21142899999998, 280.576, 197.21142899999998);
  offsetBezierCurveTo(offsetObj, 272.85942900000003, 197.21142899999998, 267.282286, 202.788572, 266.422857, 210.92571399999997);
  offsetBezierCurveTo(offsetObj, 252.288, 325.778285, 246.29028600000004, 325.357714, 128.85942900000003, 347.209143);
  offsetBezierCurveTo(offsetObj, 121.14285800000003, 348.928, 115.56571500000004, 354.066286, 115.56571400000003, 361.782857);
  offsetBezierCurveTo(offsetObj, 115.56571400000003, 370.358857, 121.14285700000003, 375.076571, 130.56000000000003, 376.356571);
  offsetBezierCurveTo(offsetObj, 247.14971400000002, 395.209142, 252.288, 398.20799999999997, 266.422857, 512.64);
  offsetBezierCurveTo(offsetObj, 267.282286, 521.216, 272.85942800000004, 526.354286, 280.576, 526.354286);
  ctx.closePath();
  offsetMoveTo(offsetObj, 571.136, 1000.777143);
  offsetBezierCurveTo(offsetObj, 582.2902859999999, 1000.777143, 590.427429, 992.64, 592.5668569999999, 981.065143);
  offsetBezierCurveTo(offsetObj, 622.9942859999999, 746.221714, 655.9999999999999, 710.637714, 888.2834289999998, 684.9280000000001);
  offsetBezierCurveTo(offsetObj, 900.2788579999998, 683.6480000000001, 908.4342859999998, 674.6331430000001, 908.4342859999998, 663.4971430000002);
  offsetBezierCurveTo(offsetObj, 908.4342859999998, 652.3428570000002, 900.2788569999998, 643.7851430000002, 888.2834289999998, 642.0662860000002);
  offsetBezierCurveTo(offsetObj, 655.9999999999998, 616.3565720000003, 622.9942859999999, 580.7908570000002, 592.5668569999998, 345.92914300000024);
  offsetBezierCurveTo(offsetObj, 590.4274279999998, 334.35428600000023, 582.2902859999998, 326.63771400000024, 571.1359999999999, 326.63771400000024);
  offsetBezierCurveTo(offsetObj, 559.9999999999999, 326.63771400000024, 551.8628569999998, 334.35428500000023, 550.1439999999999, 345.92914300000024);
  offsetBezierCurveTo(offsetObj, 519.7165709999999, 580.7908570000002, 486.2902859999999, 616.3565720000003, 254.4274289999999, 642.0662860000002);
  offsetBezierCurveTo(offsetObj, 241.99314299999992, 643.7851430000002, 233.8559999999999, 652.3611430000002, 233.8559999999999, 663.4971430000002);
  offsetBezierCurveTo(offsetObj, 233.8559999999999, 674.6514290000001, 241.99314299999992, 683.6480000000001, 254.4274289999999, 684.9280000000001);
  offsetBezierCurveTo(offsetObj, 485.85142899999994, 715.3554290000001, 517.997715, 746.6422860000001, 550.1439999999999, 981.065143);
  offsetBezierCurveTo(offsetObj, 551.8628569999998, 992.64, 559.9999999999999, 1000.777143, 571.1359999999999, 1000.777143);
  ctx.closePath();
  ctx.fill();
  ctx.restore();
}

const drawSunlightIcon = (offsetX, offsetY, size = 100) => {
  const scale = size / 1000
  const offsetObj = { offsetX, offsetY }
  ctx.save();
  ctx.miterLimit = 4;
  ctx.scale(scale, scale);
  ctx.save();
  ctx.fillStyle = colorConfig.daylight;  //填充颜色
  ctx.beginPath();
  offsetMoveTo(offsetObj, 549.302857, 84.297143);
  offsetBezierCurveTo(offsetObj, 549.302857, 64.146286, 532.150857, 46.994286, 512, 46.994286);
  offsetBezierCurveTo(offsetObj, 491.849143, 46.994286, 474.715429, 64.146286, 474.715429, 84.297143);
  offsetLineTo(offsetObj, 474.715429, 174.262857);
  offsetBezierCurveTo(offsetObj, 474.715429, 194.413714, 491.86742899999996, 211.565714, 512.018286, 211.565714);
  offsetBezierCurveTo(offsetObj, 532.132572, 211.565714, 549.284572, 194.41371400000003, 549.284571, 174.262857);
  ctx.closePath();
  offsetMoveTo(offsetObj, 723.712, 247.570286);
  offsetBezierCurveTo(offsetObj, 709.577143, 262.144, 709.577143, 286.153143, 723.712, 300.288);
  offsetBezierCurveTo(offsetObj, 738.285714, 314.861714, 761.856, 315.282286, 776.850286, 300.288);
  offsetLineTo(offsetObj, 840.7222859999999, 236.43428600000001);
  ctx.translate(813.9004234005793, 209.856);
  ctx.rotate(0);
  ctx.arc(offsetX, offsetY, 37.76, 0.7808368481847625, -0.7808368481847627, 1);
  ctx.rotate(0);
  ctx.translate(-813.9004234005793, -209.856);
  offsetBezierCurveTo(offsetObj, 826.5691429999999, 169.142857, 802.5782859999999, 169.142857, 788.0045709999999, 183.277714);
  ctx.closePath();
  offsetMoveTo(offsetObj, 247.14971400000002, 300.288);
  offsetBezierCurveTo(offsetObj, 261.284571, 314.861714, 285.293714, 314.861714, 299.849143, 300.288);
  offsetBezierCurveTo(offsetObj, 314.002286, 286.573714, 314.002286, 261.705143, 300.288, 247.570286);
  offsetLineTo(offsetObj, 236.43428600000001, 183.277714);
  offsetBezierCurveTo(offsetObj, 222.72000000000003, 169.56342800000002, 198.290286, 169.142857, 183.71657100000002, 183.277714);
  offsetBezierCurveTo(offsetObj, 169.56342800000002, 197.430857, 169.56342800000002, 221.860571, 183.277714, 235.995429);
  ctx.closePath();
  offsetMoveTo(offsetObj, 512, 293.430857);
  offsetBezierCurveTo(offsetObj, 392.429714, 293.430857, 293.430857, 392.429714, 293.430857, 512);
  offsetBezierCurveTo(offsetObj, 293.430857, 631.570286, 392.411429, 731.008, 512, 731.008);
  offsetBezierCurveTo(offsetObj, 631.149714, 731.008, 730.148571, 631.570286, 730.148571, 512);
  offsetBezierCurveTo(offsetObj, 730.148571, 392.411429, 631.1497139999999, 293.430857, 511.99999999999994, 293.430857);
  ctx.closePath();
  offsetMoveTo(offsetObj, 938.422857, 549.284571);
  offsetBezierCurveTo(offsetObj, 958.573714, 549.284571, 975.725714, 532.150857, 975.725714, 512);
  offsetBezierCurveTo(offsetObj, 975.725714, 491.849143, 958.573714, 474.697143, 938.422857, 474.697143);
  offsetLineTo(offsetObj, 848.859429, 474.697143);
  offsetBezierCurveTo(offsetObj, 828.708572, 474.697143, 811.556572, 491.84914299999997, 811.556571, 512);
  offsetBezierCurveTo(offsetObj, 811.55657, 532.1508570000001, 828.708571, 549.284571, 848.859429, 549.284571);
  ctx.closePath();
  offsetMoveTo(offsetObj, 85.577143, 474.715429);
  offsetBezierCurveTo(offsetObj, 65.426286, 474.715429, 48.274286000000004, 491.84914299999997, 48.274286000000004, 512);
  offsetBezierCurveTo(offsetObj, 48.274286000000004, 532.1508570000001, 65.426286, 549.284571, 85.577143, 549.284571);
  offsetLineTo(offsetObj, 175.14057100000002, 549.284571);
  offsetBezierCurveTo(offsetObj, 195.29142800000002, 549.284571, 212.44342800000004, 532.150857, 212.44342900000004, 512);
  offsetBezierCurveTo(offsetObj, 212.44343000000003, 491.849143, 195.29142900000005, 474.697143, 175.14057100000002, 474.697143);
  ctx.closePath();
  offsetMoveTo(offsetObj, 776.411429, 724.114286);
  ctx.translate(750.0525715, 751.5335774354928);
  ctx.rotate(0);
  ctx.arc(offsetX, offsetY, 38.034286, -0.805114285038535, -2.336478368551258, 1);
  ctx.rotate(0);
  ctx.translate(-750.0525715, -751.5335774354928);
  offsetBezierCurveTo(offsetObj, 709.558857, 738.267429, 709.558857, 762.276572, 723.693714, 776.832);
  offsetLineTo(offsetObj, 788.004571, 841.142857);
  offsetBezierCurveTo(offsetObj, 802.578285, 855.2777140000001, 826.587428, 854.8571430000001, 840.722286, 840.722286);
  offsetBezierCurveTo(offsetObj, 855.296, 826.569143, 855.296, 802.578286, 840.722286, 788.004571);
  ctx.closePath();
  offsetMoveTo(offsetObj, 183.25942899999995, 787.565714);
  offsetBezierCurveTo(offsetObj, 168.70399999999995, 801.700571, 168.70399999999995, 826.130285, 182.83885699999996, 840.265143);
  offsetBezierCurveTo(offsetObj, 196.99199999999996, 854.418286, 221.42171399999995, 854.838857, 235.97714299999996, 840.704);
  offsetLineTo(offsetObj, 299.8308569999999, 776.832);
  offsetBezierCurveTo(offsetObj, 313.9839999999999, 762.697143, 313.9839999999999, 738.688, 300.2697139999999, 724.132571);
  offsetBezierCurveTo(offsetObj, 286.1165709999999, 709.979428, 261.6868569999999, 709.979428, 247.1314289999999, 724.132571);
  ctx.closePath();
  offsetMoveTo(offsetObj, 549.266286, 849.700571);
  offsetBezierCurveTo(offsetObj, 549.266286, 829.549714, 532.132572, 812.397714, 511.981714, 812.397714);
  offsetBezierCurveTo(offsetObj, 491.83085600000004, 812.397714, 474.697143, 829.549714, 474.697143, 849.700571);
  offsetLineTo(offsetObj, 474.697143, 939.702857);
  offsetBezierCurveTo(offsetObj, 474.697143, 959.835428, 491.84914299999997, 976.987428, 512, 976.987429);
  offsetBezierCurveTo(offsetObj, 532.114286, 976.987429, 549.266286, 959.835429, 549.266286, 939.684571);
  ctx.closePath();
  ctx.fill();
  ctx.restore();
}

// 绘制图标偏移量
const offsetMoveTo = ({ offsetX, offsetY }, x, y) => {
  ctx.moveTo(x + offsetX, y + offsetY);
};
const offsetBezierCurveTo = ({ offsetX, offsetY }, x, y, x1, y1, x2, y2) => {
  ctx.bezierCurveTo(
    x + offsetX, y + offsetY,
    x1 + offsetX, y1 + offsetY,
    x2 + offsetX, y2 + offsetY
  );
}
const offsetLineTo = ({ offsetX, offsetY }, x, y) => {
  ctx.lineTo(x + offsetX, y + offsetY);
}
const offsetTranslate = ({ offsetX, offsetY }, x, y) => {
  ctx.translate(x + offsetX, y + offsetY);
}
</script>

<style lang="scss" scoped>
.canvas {
  image-rendering: high-quality;
  // image-rendering: pixelated;
  // image-rendering: crisp-edges;
}
</style>