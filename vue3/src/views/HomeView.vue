<script setup>
import { ref } from 'vue';
import Psd from '@webtoon/psd';

const canvasRef = ref(null);
const layerArr = ref([]);
const onInputFileChange = async (e) => {
  const imgRegExp = /.(jpeg|png)$/i;
  const file = e.currentTarget.files[0];
  // console.log(file)
  if (imgRegExp.test(file.type)) {
    //jpeg, png
  } else {
    if (/.psd$/.test(file.name)) {
      //psd
      const result = await file.arrayBuffer();
      const psdFile = Psd.parse(result);
      for (let i = 0; i < psdFile.layers.length; i++) {
        layerArr.value.push({
          width: psdFile.layers[i].width,
          height: psdFile.layers[i].height,
          name: psdFile.layers[i].layerFrame.layerProperties.name
        });
        const compositeBuffer = await psdFile.layers[i].composite();
        const imageData = new ImageData(
          compositeBuffer,
          psdFile.layers[i].width,
          psdFile.layers[i].height
        );
        const context = canvasRef.value[i].getContext('2d');
        canvasRef.value[i].width = psdFile.layers[i].width;
        canvasRef.value[i].height = psdFile.layers[i].height;
        context.putImageData(imageData, 0, 0);
      }
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
    <div v-for="(v, i) of layerArr">
      <h4>{{ v.name }}</h4>
      <canvas ref="canvasRef" />
    </div>
  </div>
</template>
<style scoped>
#wrap {
  display: flex;
  flex-direction: column;
}
</style>
