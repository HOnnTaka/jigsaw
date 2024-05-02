<script setup>
import { RouterLink, RouterView } from "vue-router";
import { ref, onMounted } from "vue";
const blocks = ref([]);
onMounted(() => {
  for (let i = 0; i < 20; i++) {
    blocks.value.push({
      left: Math.random() * 100,
      speed: Math.random() * (2 - 0.5) + 0.5,
    });
  }
});

const bgUrl = ref("");
const hide = ref(true);
const onImageChange = url => {
  if (url) {
    hide.value = false;
    bgUrl.value = url;
  } else {
    hide.value = true;
  }
};
</script>

<template>
  <div class="app">
    <div class="background">
      <div
        v-for="(block, index) in blocks"
        :key="index"
        class="block"
        :style="{
          left: `calc(${block.left}vw - 100px)`,
          animationDuration: `${block.speed}s`,
        }"
      ></div>
    </div>
    <div class="background2" :style="{ backgroundImage: `url(${bgUrl})`, opacity: hide ? 0 : 1 }"></div>
    <div class="app-main">
      <router-view v-slot="{ Component }">
        <transition name="animate" mode="out-in">
          <component :is="Component" @imageChange="onImageChange" />
        </transition>
      </router-view>
    </div>
    <div class="app-footer">
      <RouterLink style="flex: 1" to="/" replace>返回首页</RouterLink>
      <p style="flex: 1; text-align: center">21级马裕博</p>
      <a style="flex: 1; text-align: right" href="https://github.com/HOnnTaka/jigsaw/tree/gh-pages" target="_blank">github</a>
    </div>
  </div>
</template>

<style scoped>
.background {
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  overflow: hidden;
  pointer-events: none;
}
.background2 {
  transition: all 0.5s;
  filter: blur(7px) opacity(0.4);
  background-size: cover;
  position: fixed;
  pointer-events: none;
  inset: 0;
  width: 100vw;
  height: 100vh;
}
.block {
  position: absolute;
  top: 0;
  width: 150px;
  height: 150px;
  background-color: #e3f1fa7a;
  animation: dropDown linear infinite;
}

@keyframes dropDown {
  0% {
    transform: translateY(-200px);
  }
  100% {
    transform: translateY(100vh);
  }
}
.app-main {
  position: fixed;
  width: 100%;
  height: 95vh;
}
.app-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 2vmin;
  font-size: 14px;
  color: #999;
  height: 5vh;
  position: fixed;
  bottom: 0;
  width: 100%;
  z-index: 1000;
}

.animate-enter-active,
.animate-leave-active {
  transition: transform 0.85s;
}

.animate-enter-from,
.animate-leave-to {
  transform: translateY(100vh);
}

@media (min-width: 1024px) {
}
</style>
