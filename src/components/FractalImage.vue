<template>
  <p style="border: 5px;">
    <canvas :id="imageId" :width="width" :height="height"></canvas>
  </p>
</template>

<script>
// import Complex from "complex.js";

export default {
  name: "FractalImage",
  imageData: null,
  props: {
    width: Number,
    height: Number,
    maxIteration: { default: 64, type: Number },
    imageId: String
  },
  context: function () {
      let canvas = document.getElementById(this.imageId);
      return canvas.getContext('2d');
  },
  mounted() {
    let canvas = document.getElementById(this.imageId);
    let context = canvas.getContext('2d');

    this.imageData = context.getImageData(0, 0, this.width, this.height);

    let top_left = [-1.5, 0.5]
    let bottom_right = [-0.5, -0.5]
    let increment_x = Math.abs(bottom_right[0] - top_left[0]) / this.width
    let increment_y = Math.abs(bottom_right[1] - top_left[1]) / this.height

    for (let y = top_left[1], row = 0; y > bottom_right[1]; y -= increment_y, row++) {
      for (let x = top_left[0], col = 0; x < bottom_right[0]; x += increment_x, col++) {
        let iterCount = this.calculateMandel(x, y)
        this.setPixel(col, row, iterCount / this.maxIteration * 256, 0, 0, 255);
      }
    }
    context.putImageData(this.imageData, 0, 0);
  },
  methods: {
    calculateMandel: function (x, y) {
      let a = 0, aa = 0, b = 0, bb = 0, m;
      let n = 0 // count iteration
      do {
        b = 2 * a * b + y; // imaginary part of  z(n+1) = z(n)^2 + c
        a = aa - bb + x; // real part of  z(n+1) = z(n)^2 + c
        m = (aa = a * a) + (bb = b * b); // calculate distance to origin
      } while (++n < this.maxIteration && m < 4);
      return n
    },
    setPixel: function (x, y, r, g, b, a) {
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