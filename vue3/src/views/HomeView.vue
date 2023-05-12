<script setup>
import { ref } from 'vue';
import Psd from '@webtoon/psd';

const canvasRef = ref(null);
const konvaStageRef = ref(null);
const konvaLayerRef = ref(null);
const konvaStage = ref({ width: null, height: null });
const konvaImage = ref(null);
const konvaLineColor = ref('black');
const konvaLineWidth = ref(5);
const konvaLineComposite = ref('source-over');
const konvaLineArr = ref([]);

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

let isPress = false;
let isMove = false;
const onStageMouseDown = () => {
  isPress = true;
  isMove = false;
  const { x, y } = konvaStageRef.value.getNode().getPointerPosition();
  konvaLineArr.value.push({
    points: [x, y],
    stroke: konvaLineColor.value,
    strokeWidth: konvaLineWidth.value,
    globalCompositeOperation: konvaLineComposite.value
  });
};
const onStageMouseMove = () => {
  if (isPress) {
    isMove = true;
    const { x, y } = konvaStageRef.value.getNode().getPointerPosition();
    konvaLineArr.value[konvaLineArr.value.length - 1] = {
      points: [...konvaLineArr.value[konvaLineArr.value.length - 1].points, x, y],
      stroke: konvaLineColor.value,
      strokeWidth: konvaLineWidth.value,
      globalCompositeOperation: konvaLineComposite.value
    };
  }
};
const onStageMouseUp = () => {
  isPress = false;
  if (isMove === false) konvaLineArr.value.pop();
  isMove = false;
};
const onStageMouseLeave = () => {
  isPress = false;
};
const onClickDrawButton = () => {
  konvaLineComposite.value = 'source-over';
};
const onClickEraserButton = () => {
  konvaLineComposite.value = 'destination-out';
};
</script>
<template>
  <div id="wrap">
    <input type="file" @change="onInputFileChange" />
    <button @click="onClickDrawButton">드로잉</button>
    <button @click="onClickEraserButton">지우개</button>
    <canvas class="tempCanvas" ref="canvasRef" />
    <v-stage
      @mousemove="onStageMouseMove"
      @mouseup="onStageMouseUp"
      @mousedown="onStageMouseDown"
      @mouseleave="onStageMouseLeave"
      ref="konvaStageRef"
      :config="konvaStage"
    >
      <v-layer>
        <v-image :config="{ image: konvaImage }" />
      </v-layer>
      <v-layer ref="konvaLayerRef">
        <v-line
          v-for="v in konvaLineArr"
          :config="{
            points: v.points,
            stroke: v.stroke,
            strokeWidth: v.strokeWidth,
            globalCompositeOperation: v.globalCompositeOperation
          }"
        />
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
