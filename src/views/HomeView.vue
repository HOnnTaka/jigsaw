<template>
  <div id="home">
    <div class="loading" v-if="hide">{{ loadingText }}</div>
    <!-- 关卡容器居中轮播 -->
    <transition-group
      @mouseenter="clearTimer"
      @mouseleave="startTimer"
      @touchstart="handleTouchstart"
      @touchmove="handleTouchMove"
      @touchend="handleTouchEnd"
      name="move"
      tag="div"
      class="container"
      :class="{ hide: hide }"
    >
      <div
        class="item"
        v-for="(img, index) in images"
        :key="img"
        @click="starGame(index)"
      >
        <img :src="img" alt="" @load="imageLoaded(index)" />
      </div>
    </transition-group>
    <div class="btnBox" v-if="!hide" :class="{ hide: hide }">
      <div
        class="lt"
        @mousemove="clearTimer"
        @mouseleave="startTimer"
        @click="prevImg"
      >
        &lt;
      </div>
      <div
        class="rt"
        @mousemove="clearTimer"
        @mouseleave="startTimer"
        @click="nextImg"
      >
        &gt;
      </div>
    </div>
    <div class="hint">
      <span>点击图片开始游戏 OR </span>
      <span @click="uploadImage">上传图片</span>
    </div>
  </div>
</template>

<script setup>
import { useRouter } from "vue-router";
import {
  ref,
  onMounted,
  watch,
  onBeforeUnmount,
  computed,
  defineEmits,
} from "vue";

const router = useRouter();
const images = ref([]);
import img1 from "@/assets/images/img1.jpg";
import img5 from "@/assets/images/img5.jpg";
import img2 from "@/assets/images/img2.jpg";
import img3 from "@/assets/images/img3.jpg";
import img4 from "@/assets/images/img4.jpg";
const originImages = [img1, img5, img2, img3, img4];
const loadCount = ref(0);
const hide = ref(true);

const loadingText = computed(() => {
  return (
    "加载默认图片中..." +
    Math.floor((loadCount.value / originImages.length) * 100) +
    "%"
  );
});

const emit = defineEmits(["imageChange"]);
watch(images, () => {
  if (!hide.value) {
    emit("imageChange", images.value[2]);
  }
});

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
      images.value.push(dataURL);
    } catch (error) {
      alert("加载失败，请重试:" + error);
      console.error("Failed to load image:", img, error);
    }
  }
  // images.value = processedImages;
};

onMounted(async () => {
  await loadImages();
  emit("imageChange", images.value[2]);
});

const imageLoaded = index => {
  loadCount.value++;
  if (loadCount.value === originImages.length) {
    setTimeout(() => {
      hide.value = false;
      startTimer();
    }, 500);
  }
};

// 轮播
const timer = ref(null);
const clearTimer = () => {
  clearInterval(timer.value);
  timer.value = null;
};
const startTimer = () => {
  if (!timer.value) {
    timer.value = setInterval(nextImg, 2500);
  }
};

// 节流
const looping = ref(false);
const nextImg = () => {
  if (looping.value) {
    return;
  }
  looping.value = true;
  setTimeout(() => {
    looping.value = false;
  }, 500);
  const newImages = [...images.value];
  newImages.push(newImages.shift());
  images.value = newImages;
};
const prevImg = () => {
  if (looping.value) {
    return;
  }
  looping.value = true;
  setTimeout(() => {
    looping.value = false;
  }, 500);
  const newImages = [...images.value];
  newImages.unshift(newImages.pop());
  images.value = newImages;
};
// 移动端拖动
const handleTouchstart = e => {
  clearTimer();
};
const handleTouchMove = e => {
  clearTimer();
};
const handleTouchEnd = e => {
  const { clientX, clientY } = e.changedTouches[0];
  const { left, right, top, bottom } = e.target.getBoundingClientRect();
  if (clientX < left + 100) {
    nextImg();
  } else if (clientX > right - 100) {
    prevImg();
  }
};

// 单击图片
const starGame = index => {
  localStorage.setItem("currentImg", images.value[index]);
  router.push({ name: "game" });
};

// 上传图片
const uploadImage = async () => {
  const input = document.createElement("input");
  input.type = "file";
  input.accept = "image/*";
  input.onchange = async () => {
    clearTimer();
    const file = input.files[0];
    if (file.type.indexOf("image") === -1) {
      alert("请选择图片文件");
      return;
    }
    if (file) {
      const reader = new FileReader();
      reader.readAsDataURL(file);
      reader.onload = async () => {
        const originBase64 = reader.result;
        //   对图片进行裁剪
        const dataURL = await loadImage(originBase64);
        await localStorage.setItem("currentImg", dataURL);
        // emit("imageChange", dataURL);
        router.push({ name: "game" });
      };
    }
  };
  input.onerror = e => {
    alert("上传失败，请重试：" + e);
    console.error("上传失败，请重试：" + e);
  };
  input.onclick = () => {
    clearTimer();
  };
  input.oncancel = () => {
    startTimer();
  };
  input.click();
};

onBeforeUnmount(() => {
  clearTimer();
  emit("imageChange", null);
});
</script>

<style scoped>
#home {
  width: 100%;
  height: 100%;
}
.loading {
  display: flex;
  align-items: center;
  position: absolute;
  justify-content: center;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  transition: all 0.3s ease-in-out;
  width: 90%;
  height: 50vmin;
  font-size: 10vmin;
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
  -webkit-user-select: none;
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
