// PuzzlePiece.vue
<template>
  <div ref="piece" @mousedown="onDragStart" :style="{left: x + 'px', top: y + 'px'}"></div>
</template>

<script setup>
import { defineProps } from 'vue';
const props = defineProps({
  x: { type: Number, required: true },
  y: { type: Number, required: true },
  order: { type: Number, required: true },
});

const onDragStart = (event) => {
  event.preventDefault();
  const piece = event.target;
  const { x, y } = piece.getBoundingClientRect();
  const { left, top } = piece.style;
  const offsetX = event.clientX - x;
  const offsetY = event.clientY - y;
  const move = (event) => {
    piece.style.left = event.clientX - offsetX + 'px';
    piece.style.top = event.clientY - offsetY + 'px';
  };
  const up = () => {
    document.removeEventListener('mousemove', move);
    document.removeEventListener('mouseup', up);
  };
  document.addEventListener('mousemove', move);
  document.addEventListener('mouseup', up);
};

</script>
