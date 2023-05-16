# canvasPractice

&lt;canvas/> 공부하는 레포

- @webtoon/psd 라이브러리 -> psd에서 imageData를 추출하여 canvas에 주입

  (ImageData생성자를 통해 imageData를 생성할 경우 입력 데이터의 width,height는 변형되면 안됨)

- canvas.toDataURL('image/jpeg')를 사용하여 canvas에 그려진 요소를 image/jpeg형식의 dataURL을 반환함

- toDataURL()메소드는 canvas태그에만 사용이 가능하므로 html2canvas 라이브러리를 사용하여 원하는 DOM의 캡처 결과를 canvas에 주입한 뒤 응용활용가능함

- konva.js === stage > layer > line,shape,text,tag + label

  stage -> canvas의 wrap

  layer -> wrap안의 canvas

  line,shape,text -> layer 내부의 선,텍스트,도형

  label -> html의 label 태그와 동일 label 내부의 line,shape,text,tag를 하나로 묶음
