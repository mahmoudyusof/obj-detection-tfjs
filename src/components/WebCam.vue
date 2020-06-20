<template>
  <div>
    <video autoplay ref="vid" width="800" height="600" style="display: none;"></video>
    <canvas ref="can" width="800" height="600"></canvas>
  </div>
</template>

<script>
const tf = require("@tensorflow/tfjs");
const cocoSsd = require("@tensorflow-models/coco-ssd");
export default {
  name: "web-cam",
  methods: {
    draw(context) {
      context.drawImage(this.$refs.vid, 0, 0, 800, 600);
      setTimeout(this.draw, 20, context);
    },
    detect(model, vid, context) {
      model.detect(vid).then(res => {
        res.forEach(detected => {
          let [x, y, w, h] = detected.bbox;
          context.beginPath();
          context.rect(x, y, w, h);
          context.stroke();
          console.log(x, y, w, h);
        });
      });

      setTimeout(this.detect, 20, model, vid, context);
    }
  },
  mounted() {
    navigator.getUserMedia =
      navigator.getUserMedia ||
      navigator.webkitGetUserMedia ||
      navigator.mozGetUserMedia;
    navigator.getUserMedia(
      { audio: false, video: { width: 800, height: 600 } },
      stream => {
        this.$refs.can.width = 800;
        this.$refs.can.height = 600;
        var context = this.$refs.can.getContext("2d");
        context.width = 800;
        context.height = 600;
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
      },
      error => {
        console.log(error);
        this.$emit();
      }
    );
  }
};
</script>

<style>
</style>