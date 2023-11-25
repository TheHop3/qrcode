<template>
  <div class="reader">
    <h1>二维码解码</h1>
    <hr />
    <button class="button" @click="media">扫一扫</button>
    <button class="button">
      <input type="file" @change="upload" />从相册选择
    </button>
    <textarea class="result" v-model="result" placeholder="二维码识别内容" />
    <div class="image">
      <img :src="currentImg" alt="当前识别的二维码" />
    </div>
    <canvas class="canvas" ref="canvas"></canvas>
  </div>
</template>

<script>
import jsQR from "jsqr";
import "jimp";
import qrcodeImg from "../assets/qrcode.png";
export default {
  name: "ReaderView",
  data() {
    return {
      result: "陪念念长大",
      currentImg: qrcodeImg,
      isAnimation: true,
      timer: null,
      cvsele: Object,
      canvas: Object,
      audio: Object,
      video: Object,
    }
  },
  created() {

  },
  methods: {
    cance() {
      this.isAnimation = false;
      cancelAnimationFrame(this.timer);
      setTimeout(() => {
        this.cvsele.style.display = "none";
      }, 1000);
    },
    upload(event) {
      this.cance();
      const file = event.target.files[0];
      const createObjectURL = (window.URL || window.webkitURL).createObjectURL || window.createObjectURL;
      this.result = "";
      this.currentImg = createObjectURL(file);
      const fReader = new FileReader();
      fReader.readAsDataURL(file); // base64
      fReader.onload = (e) => {
        this.currentImg = e.target.result;
        e.target.result && Jimp.read(e.target.result).then(async (res) => {
          const { data, width, height } = res.bitmap;
          try {
            const resolve = await jsQR(data, width, height);
            this.result = resolve.data;
          } catch (error) {
            this.result = "解码失败";
          } finally {
            console.log("识别到的内容为：", this.result);
          }
        }).catch((err) => {
          console.error("识别失败", err);
        });
      };
    },
    /**
     * 扫描
     */
    draw(begin, end) {
      this.canvas.beginPath();
      this.canvas.moveTo(begin.x, begin.y);
      this.canvas.lineTo(end.x, end.y);
      this.canvas.lineWidth = 3;
      this.canvas.strokeStyle = "red";
      this.canvas.stroke();
    },
    media() {
      this.isAnimation = true;
      this.cvsele.style.display = "block";
      navigator.getUserMedia = navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia || navigator.msGetUserMedia;
      if (navigator.mediaDevices) {
        navigator.mediaDevices.getUserMedia({
          video: { facingMode: "environment" },
        }).then(res => {
          this.video.srcObject = res;
          this.video.setAttribute("playsinline", true);
          this.video.setAttribute("webkit-playsinline", true);
          this.video.addEventListener("loadedmetadata", () => {
            this.video.play();
            this.sweep();
          });
        }).catch(err => {
          this.cance();
          alert("未识别到扫描设备");
        })
      } else if (navigator.getUserMedia) {
        navigator.getUserMedia({
          video: { facingMode: "environment" },
        }).then(res => {
          this.video.srcObject = res;
          this.video.setAttribute("playsinline", true);
          this.video.setAttribute("webkit-playsinline", true);
          this.video.addEventListener("loadedmetadata", () => {
            this.video.play();
            this.sweep();
          });
        }).catch(err => {
          this.cance();
          alert("未识别到扫描设备");
        })
      } else {
        if (navigator.userAgent.toLowerCase().match(/chrome/) && location.origin.indexOf("https://") < 0) {
          console.log("获取浏览器录音功能，因安全性问题，需要在localhost 或 127.0.0.1 或 https 下才能获取权限！");
        } else {
          this.cance();
          alert("未识别到扫描设备");
        }
      }
    },
    sweep() {
      if (this.video.readyState === this.video.HAVE_ENOUGH_DATA) {
        const { videoWidth, videoHeight } = this.video;
        this.cvsele.width = videoWidth;
        this.cvsele.height = videoHeight;
        this.canvas.drawImage(this.video, 0, 0, videoWidth, videoHeight);
        try {
          const img = this.canvas.getImageData(0, 0, videoWidth, videoHeight);
          this.currentImg = img;
          const obj = jsQR(img.data, img.width, img.height, {
            inversionAttempts: "dontInvert"
          });
          if (obj) {
           const loc = obj.location;
            this.draw(loc.topLeftCorner, loc.topRightCorner);
            this.draw(loc.topRightCorner, loc.bottomRightCorner);
            this.draw(loc.bottomRightCorner, loc.bottomLeftCorner);
            this.draw(loc.bottomLeftCorner, loc.topLeftCorner);
            if (this.result != obj.data) {
              this.audio.play();
              this.result = obj.data;
              this.cance();
              console.info("识别结果：", obj.data);
            }
          } else {
            console.error("识别失败，请检查二维码是否正确！");
          }
        } catch (error) {
          console.error("识别失败，请检查二维码是否正确！");
        }
      }
      if (this.isAnimation) {
        this.timer = requestAnimationFrame(() => {
          this.sweep();
        });
      }
    }
  },
  mounted() {
    this.audio = new Audio("../assets/tone.mp3");
    this.video = document.createElement("video");
    this.cvsele = this.$refs.canvas;
    this.canvas = this.cvsele.getContext("2d");
  }
}
</script>

<style lang="scss">
.reader {
  .button {
    position: relative;
    margin: 20px;
    padding: 12px;
    width: 300px;
    cursor: pointer;
    color: #fff;
    background: #42b983;
    border: 1px solid #42b983;
    border-radius: 4px;
    overflow: hidden;
    input {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      opacity: 0;
      cursor: pointer;
    }
  }
  .result {
    width: 70%;
    box-sizing: border-box;
    padding: 10px;
    border: 1px solid gray;
    border-radius: 8px;
    font-size: 16px;
  }
  .image {
    width: 200px;
    height: 200px;
    margin: 20px auto;
    img {
      width: 100%;
      height: 100%;
    }
  }
  .canvas {
    display: none;
    box-sizing: border-box;
    position: fixed;
    top: 0;
    right: 0;
    left: 0;
    bottom: 0;
    width: 100vw;
    height: 100vh;
  }
}
</style>

