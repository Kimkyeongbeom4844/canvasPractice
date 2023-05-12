<script setup>
import { ref } from 'vue';
import Psd from '@webtoon/psd';

const canvasRef = ref(null);
const konvaStage = ref({ width: null, height: null });
const konvaImage = ref(null);

const onInputFileChange = async (e) => {
  const imgRegExp = /.(jpeg|png)$/i;
  const file = e.currentTarget.files[0];
  // console.log(file)
  if (imgRegExp.test(file.type)) {
    //jpeg, png
  } else {
    if (/.psd$/.test(file.name)) {
      //psd
      const psdFile = Psd.parse(await file.arrayBuffer());
      const context = canvasRef.value.getContext('2d');
      const compositeBuffer = await psdFile.composite();
      const imageData = new ImageData(compositeBuffer, psdFile.width, psdFile.height);
      canvasRef.value.width = psdFile.width;
      canvasRef.value.height = psdFile.height;
      context.putImageData(imageData, 0, 0);
      const image = new Image();
      image.onload = () => {
        const canvasWidth = 500;
        image.width = canvasWidth;
        image.height = canvasWidth * (image.naturalHeight / image.naturalWidth);
        konvaStage.value.width = image.width;
        konvaStage.value.height = image.height;
        konvaImage.value = image;
      };
      image.src = canvasRef.value.toDataURL('image/jpeg');
    } else {
      alert('파일 업로드 형식은 jpeg, png, psd만 가능합니다');
      e.currentTarget.value = null;
    }
  }
};
</script>
<template>
  <div id="wrap">
    <input type="file" @change="onInputFileChange" />
    <canvas class="tempCanvas" ref="canvasRef" />
    <v-stage :config="konvaStage">
      <v-layer>
        <v-image :config="{ image: konvaImage }" />
      </v-layer>
    </v-stage>
  </div>
</template>
<style scoped>
#wrap {
  display: flex;
  flex-direction: column;
}
.tempCanvas {
  display: none;
}
</style>
