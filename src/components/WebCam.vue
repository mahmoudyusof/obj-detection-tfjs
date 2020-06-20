<template>
  <video autoplay ref="vid" width="1280" height="720"></video>
</template>

<script>
export default {
  name: "web-cam",
  mounted() {
    navigator.getUserMedia =
      navigator.getUserMedia ||
      navigator.webkitGetUserMedia ||
      navigator.mozGetUserMedia;
    navigator.getUserMedia(
      { audio: false, video: { width: 1280, height: 720 } },
      stream => {
        this.$refs.vid.srcObject = stream;
        this.$refs.vid.onloadedmetadata = e => {
          video.play();
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