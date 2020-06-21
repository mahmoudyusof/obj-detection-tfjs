<template>
  <div>
    <video autoplay ref="vid" :width="width" :height="height" style="display: none;"></video>
    <canvas ref="can" :width="width" :height="height"></canvas>
  </div>
</template>

<script>
const tf = require("@tensorflow/tfjs");
const cocoSsd = require("@tensorflow-models/coco-ssd");
export default {
  name: "web-cam",
  data: {
    width: 800,
    height: 600
  },
  methods: {
    draw(context) {
      context.drawImage(
        this.$refs.vid,
        0,
        0,
        window.innerWidth,
        window.innerHeight
      );
      setTimeout(this.draw, 20, context);
    },
    detect(model, vid, context) {
      model.detect(vid).then(res => {
        res.forEach(detected => {
          let [x, y, w, h] = detected.bbox;
          context.strokeStyle = "blue";
          context.beginPath();
          context.rect(x, y, w, h);
          context.stroke();
          context.fillStyle = "blue";
          context.fillRect(x, y - 10, 40, 10);
          context.fillStyle = "white";
          context.fillText(detected.class, x, y, 40);
        });
      });

      setTimeout(this.detect, 20, model, vid, context);
    }
  },
  mounted() {
    this.width = window.innerWidth;
    this.height = window.innerHeight;
    navigator.getUserMedia =
      navigator.getUserMedia ||
      navigator.webkitGetUserMedia ||
      navigator.mozGetUserMedia;
    navigator.mediaDevices
      .getUserMedia({
        audio: false,
        video: {
          facingMode: "environment",
          width: window.innerWidth,
          height: window.innerHeight
        }
      })
      .then(stream => {
        this.$refs.can.width = window.innerWidth;
        this.$refs.can.height = window.innerHeight;
        var context = this.$refs.can.getContext("2d");
        context.width = window.innerWidth;
        context.height = window.innerHeight;
        this.$refs.vid.srcObject = stream;
        this.$refs.vid.onplay = () => {
          this.draw(context);
        };
        this.$refs.vid.onloadedmetadata = e => {
          this.$refs.vid.play();
          cocoSsd.load().then(model => {
            this.detect(model, this.$refs.vid, context);
          });
        };
      })
      .catch(error => {
        console.log(error);
      });
  }
};
</script>

<style>
</style>