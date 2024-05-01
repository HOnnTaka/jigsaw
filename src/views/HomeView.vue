<template>
  <div id="home">
    <!-- 关卡容器居中轮播 -->
    <transition-group @mousemove="clearTimer" @mouseleave="startTimer" name="move" tag="div" class="container" :class="{ hide: hide }">
      <div class="item" v-for="(img, index) in images" :key="img" @click="starGame(index)">
        <img :src="img" alt="" />
      </div>
    </transition-group>
    <div class="btnBox" :class="{ hide: hide }">
      <div class="lt" @mousemove="clearTimer" @mouseleave="startTimer" @click="prevImg">&lt;</div>
      <div class="rt" @mousemove="clearTimer" @mouseleave="startTimer" @click="nextImg">&gt;</div>
    </div>
    <div class="hint">
      <span>点击图片开始游戏 OR </span>
      <span @click="uploadImage">上传图片</span>
    </div>
  </div>
</template>

<script setup>
import { useRouter } from "vue-router";
import { ref, onMounted } from "vue";

const router = useRouter();
const images = ref([]);
import img1 from "@/assets/images/img1.png";
import img5 from "@/assets/images/img5.jpg";
import img2 from "@/assets/images/img2.png";
import img3 from "@/assets/images/img3.jpg";
import img4 from "@/assets/images/img4.png";
const originImages = [img1, img5, img2, img3, img4];
const hide = ref(true);

const loadImage = async img => {
  return new Promise((resolve, reject) => {
    const canvas = document.createElement("canvas");
    const ctx = canvas.getContext("2d");
    const imgObj = new Image();
    imgObj.src = img;
    imgObj.onload = function () {
      const width = imgObj.width;
      const height = imgObj.height;
      const min = Math.min(width, height);
      const max = Math.max(width, height);
      // 居中裁剪为正方形
      canvas.width = min;
      canvas.height = min;
      if (width > height) {
        ctx.drawImage(imgObj, (width - min) / 2, 0, min, min, 0, 0, min, min);
      } else {
        ctx.drawImage(imgObj, 0, (height - min) / 2, min, min, 0, 0, min, min);
      }
      //    处理图片压缩
      const quality = 0.1;
      const base64 = canvas.toDataURL("image/jpeg", quality);
      resolve(base64);
    };
    imgObj.onerror = function (error) {
      reject(error);
    };
  });
};

const loadImages = async () => {
  const processedImages = [];
  for (const img of originImages) {
    try {
      const dataURL = await loadImage(img);
      processedImages.push(dataURL);
    } catch (error) {
      // 处理图片加载失败的情况
      console.error("Failed to load image:", img, error);
      // 可以选择设置默认图片或其他处理方式
    }
  }
  images.value = processedImages;
};

onMounted(async () => {
  await loadImages();
});

const timer = ref(null);
const clearTimer = () => {
  clearInterval(timer.value);
};
const startTimer = () => {
  timer.value = setInterval(nextImg, 2000);
};
setTimeout(() => {
  hide.value = false;
  startTimer();
}, 1000);
const nextImg = () => {
  const newImages = [...images.value];
  newImages.push(newImages.shift());
  images.value = newImages;
};
const prevImg = () => {
  const newImages = [...images.value];
  newImages.unshift(newImages.pop());
  images.value = newImages;
};
const starGame = index => {
  localStorage.setItem("currentImg", images.value[index]);
  router.push({ name: "game" });
};

const uploadImage = async () => {
  // 上传图片逻辑
  //   打开文件选择框
  const input = document.createElement("input");
  input.type = "file";
  input.accept = "image/*";
  input.onchange = async () => {
    const file = input.files[0];
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = async () => {
      const originBase64 = reader.result;
      //   对图片进行裁剪
      const dataURL = await loadImage(originBase64);
      localStorage.setItem("currentImg", dataURL);
        router.push({ name: "game" });
    };
  };
  input.click();
};
</script>

<style scoped>
#home {
  width: 100%;
  height: 100%;
}
/* 轮播图容器 */
.container {
  display: flex;
  align-items: center;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  transition: all 0.3s ease-in-out;
  width: 90%;
  height: 50vmin;
}
/* 轮播图项 */
.item {
  cursor: pointer;
  position: absolute;
  width: 50vmin;
  height: 50vmin;
  left: 50%;
  transform: translateX(-50%) scale(0);
  z-index: 1;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.3);
  border-radius: 10px;
  transition: all 0.5s ease-in-out;
}

.item:nth-child(2) {
  left: 25%;
  transform: translateX(-50%) scale(0.8);
  z-index: 2;
}
.item:nth-child(3) {
  left: 50%;
  transform: translateX(-50%) scale(1.2);
  z-index: 3;
}
.item:nth-child(4) {
  left: 75%;
  transform: translateX(-50%) scale(0.8);
  z-index: 2;
}

.item img {
  transition: all 0.3s ease-in-out;
  width: 50vmin;
  height: 50vmin;
  border-radius: 10px;
  user-select: none;
  -webkit-user-drag: none;
}
.hint {
    white-space: nowrap;
  position: absolute;
  top: calc(50% + 35vmin);
  left: 50%;
  transform: translate(-50%, -50%);
  pointer-events: none;
  font-size: 25px;
}
.hint span:nth-child(2) {
  cursor: pointer;
  text-decoration: underline;
  pointer-events: all;
}
.btnBox {
  display: flex;
  align-items: center;
  position: absolute;
  justify-content: space-between;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  transition: all 0.3s ease-in-out;
  width: 90%;
  height: 50vmax;
  pointer-events: none;
}
.btnBox .lt,
.btnBox .rt {
  transform: scaleY(2);
  cursor: pointer;
  font-size: 70px;
  color: rgba(0, 0, 0, 0.3);
  transition: all 0.3s ease-in-out;
  user-select: none;
  pointer-events: all;
}
.btnBox .lt:hover,
.btnBox .rt:hover {
  color: rgba(0, 0, 0, 0.9);
}
.hide {
  opacity: 0;
  pointer-events: none;
}
</style>
