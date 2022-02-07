<template>
  <p style="border: 5px;">
    <canvas :id="canvasId" :width="width" :height="height" @click="singleClick" @dblclick="dblClick"
      :title="title" @mousemove="showPos"></canvas>
  </p>
</template>

<script>
import Gradient from 'javascript-color-gradient';

let time = null;  // define time be null

export default {
  name: "FractalImage",
  imageData: null,
  props: {
    width: Number,
    height: Number,
    maxIteration: { default: 256, type: Number },
    canvasId: String,
    colours: { type: Array, default: () =>
         ['#612160', '#2e10e8', '#4aabe8', '#33cc87', '#BB44BB', '#AA55AA',
          '#4bbd5d', '#a2c261', '#efe65b', '#e2934c', '#558855', '#449944',
          '#c90d39', '#ac22bb', '#1d1c41', '#00DD00' ]},
    rect: { type: Array, default: () => [-1.5, 1.5, 1.5, -1.5] },
    title: {type: String, default: 'Click -> Zoom In ... Double Click -> Zoom Out'},
  },
  data() {
    return {
      gradient:  new Gradient('', this.maxIteration, this.colours)
    }
  },
  context: function () {
      let canvas = document.getElementById(this.imageId);
      return canvas.getContext('2d');
  },
  mounted() {
    // this.gradient =
    // console.log(this.colours.length)
    // this.gradient.setGradient(this.colours)
    this.gradient.setMidpoint(this.maxIteration)
    this.calculateImage()
  },
  methods: {
    singleClick($event) {
      // first clear  time
      clearTimeout(time);
      time = setTimeout(() => {
        this.calculateImage($event, true)
      }, 300);
    },
    dblClick($event) {
      clearTimeout(time)
      this.calculateImage($event, false)
    },
    calculateImage: function (event, zoomIn) {
      if (event) {
        let diffX = this.rect[2] - this.rect[0]
        let diffY = this.rect[3] - this.rect[1]
        let posX = this.rect[0] + event.layerX / this.width * diffX
        let posY = this.rect[1] + event.layerY / this.height * diffY
        console.log(event);
        console.log('Prev Rect: ' + this.rect);

        console.log('Pos: ' + [posX, posY])
        console.log('Moved Rect: ' + this.rect);
        this.rect.forEach(function (part, index) {
          this[index] = zoomIn ? this[index] / 2 : this[index] * 2;
        }, this.rect);
        console.log('New Rect: ' + this.rect);
        /* this.rect[0] += zoomIn ? posX / 2 : posX * 2 // - zoomIn ? Math.abs(this.rect[2] - this.rect[0]) / 2 : Math.abs(this.rect[2] - this.rect[0]) * 2
        this.rect[2] += zoomIn ? posX / 2 : posX * 2 // + zoomIn ? Math.abs(this.rect[2] - this.rect[0]) / 2 : Math.abs(this.rect[2] - this.rect[0]) * 2
        this.rect[1] += zoomIn ? posY / 2 : posY * 2 // - zoomIn ? Math.abs(this.rect[3] - this.rect[1]) / 2 : Math.abs(this.rect[3] - this.rect[1]) * 2
        this.rect[3] += zoomIn ? posY / 2 : posY * 2 // + zoomIn ? Math.abs(this.rect[3] - this.rect[1]) / 2 : Math.abs(this.rect[3] - this.rect[1]) * 2
        */

        diffX = this.rect[2] - this.rect[0]
        diffY = this.rect[3] - this.rect[1]
        this.rect[0] = posX - diffX / 2
        this.rect[2] = posX + diffX / 2
        this.rect[1] = posY - diffY / 2
        this.rect[3] = posY + diffY / 2

        console.log('Corr. Rect: ' + this.rect);
        console.log(zoomIn + ' - ' + event)
        // return
      }

      let canvas = document.getElementById(this.canvasId);
      let context = canvas.getContext('2d');

      this.imageData = context.getImageData(0, 0, this.width, this.height);

      let top_left = [this.rect[0] , this.rect[1]]
      let bottom_right = [this.rect[2] , this.rect[3]]
      let increment_x = Math.abs(bottom_right[0] - top_left[0]) / this.width
      let increment_y = Math.abs(bottom_right[1] - top_left[1]) / this.height

      for (let y = top_left[1], row = 0; y > bottom_right[1]; y -= increment_y, row++) {
        for (let x = top_left[0], col = 0; x < bottom_right[0]; x += increment_x, col++) {
          let iterCount = this.calculateMandel(x, y)
          let index = Math.round(iterCount / this.maxIteration * this.colours.length)
          let color = this.gradient.getArray()[index > 0 ? (index < this.gradient.getArray().length ? index : this.gradient.getArray().length - 1): 0]
          this.setPixel(col, row, color);
        }
      }
      context.putImageData(this.imageData, 0, 0);
    },
    calculateMandel: function (x, y) {
      let a = 0, aa = 0, b = 0, bb = 0, m;
      let n = 0 // count iteration
      do {
        b = 2 * a * b + y; // imaginary part of  z(n+1) = z(n)^2 + c
        a = aa - bb + x; // real part of  z(n+1) = z(n)^2 + c
        m = (aa = a * a) + (bb = b * b); // calculate distance to origin
      } while (++n < this.maxIteration && m < 10);
      return n
    },
    setPixel: function (x, y, hex) {
      var r = parseInt(hex.slice(1, 3), 16),
          g = parseInt(hex.slice(3, 5), 16),
          b = parseInt(hex.slice(5, 7), 16),
          a = hex.length > 7 ? parseInt(hex.slice(7, 9)) : 255;
      let index = (x + y * this.width) * 4;
      this.imageData.data[index] = r;
      this.imageData.data[index + 1] = g;
      this.imageData.data[index + 2] = b;
      this.imageData.data[index + 3] = a;
    }
  }
}
</script>

<style scoped>

</style>