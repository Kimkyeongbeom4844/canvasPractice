<script setup>
import { ref } from 'vue';
import Psd from '@webtoon/psd';

const canvasRef = ref(null);
const konvaStageRef = ref(null);
const konvaStage = ref(null);
const konvaImage = ref(null);
const konvaLineArr = ref([]); //선 배열
const konvaLineColor = ref('#000000'); // 선 색상
const konvaLineWidth = ref(5); // 선 굵기
const konvaLineComposite = ref('source-over'); // 그리기 모드
const konvaTransformerRef = ref(null);

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
        const canvasWidth = 690; //네이버는 width 690px
        image.width = canvasWidth;
        image.height = canvasWidth * (image.naturalHeight / image.naturalWidth);
        konvaStage.value = { width: image.width, height: image.height };
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
const onStageMouseDown = (e) => {
  // console.log(e.target, Object.getPrototypeOf(e.target).className);
  if (Object.getPrototypeOf(e.target).className == 'Text') {
    konvaTransformerRef.value.getNode().nodes([e.target.getParent()]);
    return;
  } else {
    if (e.target.attrs.name?.includes('_anchor'))
      isPress = false; //mousedown의 target이 transformer의 꼭짓점일때
    else {
      konvaTransformerRef.value.getNode().nodes([]);
      isPress = true;
      console.log(konvaStageRef.value.getNode());
    }
  }
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
  if (konvaLineArr.value[konvaLineArr.value.length - 1]?.points.length === 2)
    konvaLineArr.value.pop();
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
const onClickPopButton = () => {
  konvaLineArr.value.pop();
};
const onDragStart = (e) => {
  e.dataTransfer.dropEffect = 'copy';
};
const onDragEnd = (e) => {
  console.log(konvaStageRef.value.getNode().getPointerPosition());
};
</script>
<template>
  <div id="wrap">
    <div @dragstart="onDragStart" @dragend="onDragEnd">123</div>
    <input type="file" @change="onInputFileChange" />
    <input v-model="konvaLineColor" type="color" />
    <button @click="onClickDrawButton">드로잉</button>
    <button @click="onClickEraserButton">지우개</button>
    <button @click="onClickPopButton">뒤로가기</button>
    <canvas class="tempCanvas" ref="canvasRef" />
    <v-stage
      v-if="konvaStage"
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
      <v-layer>
        <v-line
          :config="{
            points: [100, 100, 200, 200],
            stroke: 'red',
            strokeWidth: 5,
            globalCompositeOperation: 'source-over'
          }"
        />
      </v-layer>
      <v-layer>
        <v-line
          v-for="v in konvaLineArr"
          :config="{
            points: v.points,
            stroke: v.stroke,
            strokeWidth: v.strokeWidth,
            globalCompositeOperation: v.globalCompositeOperation
          }"
        />
        <v-label :config="{ x: 100, y: 100, draggable: true }">
          <v-tag
            :config="{
              fill: 'white',
              stroke: 'black',
              strokeWidth: 2,
              pointerDirection: 'down',
              pointerWidth: 10,
              pointerHeight: 10
              // shadowColor: 'black'
              // shadowBlur: 10,
              // shadowOffsetX: 10,
              // shadowOffsetY: 10,
              // shadowOpacity: 0.5
            }"
          ></v-tag>
          <v-text
            :config="{
              text: `안녕하세요.
개발팀입니다
개발`,
              fontSize: 14,
              padding: 8
            }"
          />
        </v-label>
        <v-transformer ref="konvaTransformerRef" />
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
