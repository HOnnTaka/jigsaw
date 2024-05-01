<template>
  <div class="game-view">
    <Transition name="fade">
      <!-- 倒计时 -->
      <div v-if="count != 'hide'" class="count-down">{{ count }}</div>
    </Transition>
    <!-- 拼图框 -->

    <div ref="jigsawContainer" class="jigsaw-container">
      <!-- <img :src="croppedImage" alt="jigsaw-image" /> -->
      <div
        v-for="piece in pieces"
        :key="piece.id"
        :data-id="piece.id"
        :style="{
          top: `calc(${piece.y} * calc(90vmin / 4))`,
          left: `calc(${piece.x} * calc(90vmin / 4))`,
          backgroundPosition: `calc(-${piece.x} * calc(90vmin / 4)) calc(-${piece.y} * calc(90vmin / 4))`,
          backgroundImage: 'url(' + croppedImage + ')',
        }"
        class="piece"
        ref="piecesRef"
        @touchstart="onPieceTouchStart($event)"
        @touchmove="onPieceTouchMove($event)"
        @touchend="onPieceTouchEnd($event)"
        @mousedown="onPieceTouchStart($event)"
        @mousemove="onPieceTouchMove($event)"
        @mouseup="onPieceTouchEnd($event)"
      ></div>
    </div>
    <div ref="saveContainer" class="save-container"></div>
    <div class="jigsaw-containerBorder" style="position: absolute">
      <div v-for="i in 16"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
const router = useRouter();

const croppedImage = ref(localStorage.getItem("currentImg"));
const jigsawContainer = ref();
const piecesRef = ref();
const saveContainer = ref();

const initialPieces = [];
const piecesIsCurrect = ref([]);
for (let i = 0; i < 4; i++) {
  for (let j = 0; j < 4; j++) {
    const piece = Object.freeze({
      id: i * 4 + j,
      x: j,
      y: i,
    });
    initialPieces.push(piece);
    piecesIsCurrect.value.push({ id: i * 4 + j, isCurrect: false });
  }
}

const pieces = ref(initialPieces);

const count = ref();
const countDown = () => {
  setTimeout(() => {
    count.value = 3;
    const countDownInterval = setInterval(() => {
      if (count.value === 1) {
        count.value = "GO";
        shuffle();
        clearInterval(countDownInterval);
        setTimeout(() => (count.value = "hide"), 1000);
      } else {
        count.value--;
      }
    }, 1000);
  }, 1000);
};

// 打乱拼图
const shuffle = () => {
  //   console.dir(saveContainer.value);
  const pieceSize = piecesRef.value[0].offsetWidth;
  const startTop = saveContainer.value.offsetTop;
  const startLeft = saveContainer.value.offsetLeft;
  const endTop = startTop + saveContainer.value.offsetHeight - pieceSize - 20;
  const endLeft = startLeft + saveContainer.value.offsetWidth - pieceSize - 20;
  //   console.log(startTop, startLeft, endTop, endLeft);
  piecesRef.value.forEach(piece => {
    piece.classList.add("out");
    piece.classList.add("transition");
    const randomY = Math.floor(Math.random() * (endTop - startTop) + startTop);
    const randomX = Math.floor(Math.random() * (endLeft - startLeft) + startLeft);
    piece.style.left = `${randomX}px`;
    piece.style.top = `${randomY}px `;
    setTimeout(() => piece.classList.remove("transition"), 500);
  });
};

onMounted(() => {
  countDown();
});

let startX,
  startY,
  startLeft,
  startTop,
  currentPiece,
  currentId,
  isCorrect = false,
  zIndex = 1;
const onPieceTouchStart = event => {
  event.preventDefault();
  currentPiece = event.target;
  currentId = currentPiece.getAttribute("data-id");
  if (piecesIsCurrect.value.find(piece => piece.id == currentId).isCurrect) return;
  currentPiece.style.zIndex = zIndex++;
  currentPiece.classList.add("active");
  startX = (event.touches ? event.touches[0] : event).clientX;
  startY = (event.touches ? event.touches[0] : event).clientY;
  startLeft = currentPiece.offsetLeft || 0;
  startTop = currentPiece.offsetTop || 0;
};

const onPieceTouchMove = event => {
  event.preventDefault();
  if (!currentPiece) return;
  if (piecesIsCurrect.value.find(piece => piece.id == currentId).isCurrect) return;
  const newLeft = startLeft + (event.touches ? event.touches[0] : event).clientX - startX;
  const newTop = startTop + (event.touches ? event.touches[0] : event).clientY - startY;
  currentPiece.style.left = `${
    newLeft <= -20 ? -20 : newLeft >= window.innerWidth - currentPiece.offsetWidth ? window.innerWidth - currentPiece.offsetWidth : newLeft
  }px`;
  currentPiece.style.top = `${
    newTop <= -25 ? -25 : newTop >= window.innerHeight - currentPiece.offsetHeight - 10 ? window.innerHeight - currentPiece.offsetHeight - 10 : newTop
  }px`;
  const originSize = jigsawContainer.value.offsetWidth / 4;
  const { x, y } = pieces.value.find(piece => piece.id == currentId);
  const rangeX = [x * originSize, (x + 1) * originSize];
  const rangeY = [y * originSize, (y + 1) * originSize];

  // console.log(event.touches[0].clientX, event.touches[0].clientY);
  const clientX = (event.touches ? event.touches[0] : event).clientX - jigsawContainer.value.offsetLeft;
  const clientY = (event.touches ? event.touches[0] : event).clientY - jigsawContainer.value.offsetTop;
  // 判断是否在拼图范围内
  if (clientX >= rangeX[0] && clientX <= rangeX[1] && clientY >= rangeY[0] && clientY <= rangeY[1]) {
    currentPiece.classList.remove("out");
    isCorrect = true;
  } else {
    currentPiece.classList.add("out");
    isCorrect = false;
  }
};

const onPieceTouchEnd = () => {
  currentPiece.classList.remove("active");
  if (isCorrect) {
    const pieceisCurrect = piecesIsCurrect.value.find(piece => piece.id == currentId);
    const piece = pieces.value.find(piece => piece.id == currentId);
    pieceisCurrect.isCurrect = true;
    currentPiece.classList.remove("out");
    currentPiece.classList.add("transition");
    currentPiece.style.top = `calc(${piece.y} * calc(90vmin / 4))`;
    currentPiece.style.left = `calc(${piece.x} * calc(90vmin / 4))`;
    currentPiece.style.zIndex = 1;
    if (piecesIsCurrect.value.every(piece => piece.isCurrect)) {
      count.value = "完成！";
      setTimeout(() => {
        count.value = "hide";
        router.push({ name: "home" });
      }, 3000);
    }
  }
  currentPiece = null;
  isCorrect = false;
};
</script>

<style scoped>
.count-down {
  font-size: 50vmin;
  white-space: nowrap;
  width: 100vw;
  height: 100vh;
  position: fixed;
  inset: 0;
  color: rgba(255, 255, 255, 0.7);
  z-index: 9999;
  display: flex;
  justify-content: center;
  align-items: center;
}
.game-view {
  position: relative;
  height: 100%;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 2vmin;
  padding-bottom: 0;
}
.game-view > div {
  border-radius: 10px;
  box-shadow: 0px 0px 10px #00000033;
  background: #00000033;
}
.jigsaw-container {
  height: 90vmin;
  width: 90vmin;
  position: relative;
}
.game-view > div.jigsaw-containerBorder {
  position: fixed;
  left: 2vmin;
  height: 90vmin;
  width: 90vmin;
  position: relative;
  pointer-events: none;
  box-shadow: none;
  background: none;
  display: flex;
  flex-wrap: wrap;
  pointer-events: none;
}
.jigsaw-containerBorder div {
  border: 1px dashed rgba(121, 121, 121, 0.5);
  width: calc(90vmin / 4);
  height: calc(90vmin / 4);
}
.jigsaw-containerBorder div:nth-child(1) {
  border-top-left-radius: 10px;
}
.jigsaw-containerBorder div:nth-child(4) {
  border-top-right-radius: 10px;
}
.jigsaw-containerBorder div:nth-child(13) {
  border-bottom-left-radius: 10px;
}
.jigsaw-containerBorder div:nth-child(16) {
  border-bottom-right-radius: 10px;
}
.jigsaw-container img {
  height: 100%;
  width: 100%;
  object-fit: cover;
  border-radius: 10px;
}
.save-container {
  height: 90vmin;
  width: calc(100vw - 90vmin - 6vmin);
}

.piece {
  transition: box-shadow, transform 0.3s ease;
  cursor: -webkit-grab;
  position: absolute;
  width: calc(90vmin / 4);
  height: calc(90vmin / 4);
  border-radius: 10px;
  background-size: 90vmin 90vmin;
  background-repeat: no-repeat;
  outline: 1px solid #ffffff33;
}
.piece.transition {
  transition: all 0.5s ease;
}
.piece.active {
  cursor: -webkit-grabbing;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.3);
}
.piece.out {
  transform: scale(0.8);
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
@media screen and (max-width: 768px) {
  .game-view {
    flex-direction: column;
    padding-top: 5vmin;
  }
  .save-container {
    width: 90vmin;
    height: calc(100vh - 90vmin - 5vh - 10vmin);
  }

  .game-view > div.jigsaw-containerBorder {
    left: unset;
  }
  .count-down{
    font-size: 30vmin;
  }
}
</style>
