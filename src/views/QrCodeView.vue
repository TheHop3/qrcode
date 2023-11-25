<template>
  <div class="qrcode">
    <h1>二维码生成</h1>
    <hr />
    <div class="form">
      <textarea class="result" v-model="result" @input="createQRcode" placeholder="请输入要生成的内容"></textarea>
    </div>
    <div id="qrcode" ref="qrcode"></div>
  </div>
</template>

<script>
import QRCode from "qrcodejs2";
export default {
  name: "HomeView",
  data() {
    return {
      qrcode: null,
      result: "陪念念长大",
      qrcodeImg: "",
    }
  },
  created() {

  },
  methods: {
    // QRcode 生成二维码
    createQRcode() {
      this.qrcode.clear();
      this.qrcode.makeCode(this.result);
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.qrcode = new QRCode(this.$refs.qrcode, {
        // text: this.result,
        width: 200,
        height: 200,
        colorDark: "#000000",
        colorLight: "#ffffff",
        correctLevel: QRCode.CorrectLevel.H,
      });
      this.createQRcode();
    })
  }
};
</script>

<style lang="scss">
.result {
  width: 70%;
  box-sizing: border-box;
  padding: 10px;
  border: 1px solid gray;
  border-radius: 8px;
  font-size: 16px;
}
#qrcode {
  margin: 20px auto;
  width: 200px;
  height: 200px;
}
</style>
