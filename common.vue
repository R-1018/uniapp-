<template>
  <!-- 拍照上传图片带水印 -->
  <camera
    device-position="back"
    flash="off"
    @error="error"
    style="width: 100%; height: 300px"
  ></camera>
  <button type="primary" @tap="takePhoto">拍照</button>
  <view>预览</view>
  <image mode="widthFix" :src="src"></image>
  <canvas class="canvas" :style="canvasStyle" canvas-id="firstCanvas"></canvas>
</template>

<script>
export default {
  data() {
    return {
      socketOpen: false,
      src: "",
      canvasStyle: {},
    };
  },
  methods: {
    takePhoto() {
      const ctx = uni.createCameraContext();
      ctx.takePhoto({
        quality: "high",
        success: (res) => {
          this.src = res.tempImagePath;
          let that = this;
          uni.getImageInfo({
            src: res.tempImagePath,
            success: (ress) => {
              let date = "水印"; /** 作为水印 */
              let ctx = uni.createCanvasContext("firstCanvas"); /** 创建画布 */
              let textToWidth =
                (ress.width / 3) * 0.5; /**这里设置的是水印位置*/
              let textToHeight = (ress.height / 3) * 0.9;
              // that.canvasStyle = `{width:100%.width/3px;height:100%.height/3px}`
              //将图片src放到cancas内，宽高为图片大小
              ctx.drawImage(res.tempImagePath, 0, 0, ress.width, ress.height);
              //将声明的时间放入canvas
              ctx.setFontSize(14); //注意：设置文字大小必须放在填充文字之前，否则不生效
              ctx.setFillStyle("blue");
              ctx.fillText(date, textToWidth, textToHeight); //水印内容，位置
              // ctx.strokeText(date, ress.width, ress.height)
              uni.showLoading({
                title: "制作水印中...",
              });
              ctx.draw(() => {
                //开始制作
                //使用定时是因为制作水印需要时间，设置定时才不会出bug
                uni.canvasToTempFilePath({
                  //将画布中内容转成图片，即水印与图片合成
                  canvasId: ctx,
                  success: (res) => {
                    uni.hideLoading();
                    uni.saveImageToPhotosAlbum({
                      //保存到手机
                      filePath: res.tempFilePath,
                      success(res) {
                        console.log("1");
                      },
                    });
                  },
                });
              });
            },
          });
        },
      });
    },
  },
};
</script>
