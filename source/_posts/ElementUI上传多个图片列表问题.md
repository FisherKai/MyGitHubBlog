---
title: ElementUI上传多个图片列表问题
date: 2021-12-13 09:57:34
tags:
---

    关于在使用ElementUI的Upload组件进行多张图片上传时，需要在图片上传完成提示上传完成信息。
    以下附上demo代码

```
<template>
  <div>
    <el-upload
      class="upload-demo"
      ref="upload"
      action="https://jsonplaceholder.typicode.com/posts/"
      :on-preview="handlePreview"
      :on-remove="handleRemove"
      :file-list="obj.fileList"
      :on-success="success"
      :on-progress="progress"
      :before-upload="before"
      :auto-upload="false"
      :on-change="change"
      list-type="picture"
    >
      <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
      <el-button
        style="margin-left: 10px"
        size="small"
        type="success"
        @click="submitUpload"
        >上传到服务器</el-button
      >
      <div slot="tip" class="el-upload__tip">
        只能上传jpg/png文件，且不超过500kb
      </div>
    </el-upload>
    <div id="ys-picture"></div>
  </div>
</template>

<script>
import { Upload, Button, Loading } from "element-ui";
export default {
  components: {
    "el-upload": Upload,
    "el-button": Button,
  },
  data() {
    return {
      obj: {
        fileList: [],
        successList: [],
        fileListLength: 0,
      },
      loading: null,
    };
  },
  watch: {
    obj: {
      //深度监听，可监听到对象、数组的变化
      handler(val, oldVal) {
        console.log(
          val.successList.length,
          oldVal.successList.length,
          val.fileListLength
        );
        if (val.successList.length === val.fileListLength) {
          this.loading.close();
        }
      },
      deep: true, //true 深度监听
    },
  },
  methods: {
    submitUpload() {
      this.$refs.upload.submit();
      if (this.obj.fileListLength > 0) {
        this.loading = Loading.service({
          lock: true,
          text: "Loading",
          spinner: "el-icon-loading",
          background: "rgba(0, 0, 0, 0.7)",
        });
      }
    },
    handleRemove(file, fileList) {
      console.log("handleRemove", file, fileList);
    },
    handlePreview(file) {
      console.log("handlePreview", file);
    },
    success(response, file, fileList) {
      console.log("success", response, file, fileList);
      this.obj.successList.push(file.uid);
    },
    progress(event, file, fileList) {
      console.log("progress", event, file, fileList);
    },
    change(file, fileList) {
      console.log("change", file, fileList);
      this.obj.fileListLength = fileList.length;
      // window.localStorage.setItem("fileList", JSON.stringify(fileList));
    },
    before(file) {
      console.log("压缩前的文件", file);

      let _this = this;
      return new Promise((resolve, reject) => {
        let isLt2M = file.size / 1024 / 1024 < 0.01; // 判定图片大小是否小于10MB
        if (!isLt2M) {
          reject();
        }
        let image = new Image(),
          resultBlob = "";
        image.src = URL.createObjectURL(file);
        image.onload = () => {
          // 调用方法获取blob格式，方法写在下边
          resultBlob = _this.compressUpload(image, file);
          console.log("压缩过后的文件:", resultBlob);
          resolve(resultBlob);
        };
        image.onerror = () => {
          reject();
        };
      });
    },
    /* 图片压缩方法-canvas压缩 */
    compressUpload(image, file) {
      let canvas = document.createElement("canvas");
      let ctx = canvas.getContext("2d");
      let initSize = image.src.length;
      let { width } = image,
        { height } = image;
      canvas.width = width;
      canvas.height = height;
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(image, 0, 0, width, height);
      // document.getElementById("ys-picture").appendChild(canvas);

      // 进行最小压缩0.1
      let compressData = canvas.toDataURL(file.type || "image/jpeg", 0.1);

      // 压缩后调用方法进行base64转Blob，方法写在下边
      let blobImg = this.dataURItoBlob(compressData);
      return blobImg;
    },

    /* base64转Blob对象 */
    dataURItoBlob(data) {
      let byteString;
      if (data.split(",")[0].indexOf("base64") >= 0) {
        byteString = atob(data.split(",")[1]);
      } else {
        byteString = unescape(data.split(",")[1]);
      }
      let mimeString = data.split(",")[0].split(":")[1].split(";")[0];
      let ia = new Uint8Array(byteString.length);
      for (let i = 0; i < byteString.length; i += 1) {
        ia[i] = byteString.charCodeAt(i);
      }
      return new Blob([ia], { type: mimeString });
    },
  },
  mounted() {
  },
};
</script>

<style>
</style>
```